# Codex Repository Architecture

**English** | [中文](architecture_cn.md)

High-level overview of the [openai/codex](https://github.com/openai/codex) repository — what it does, how it is built, how a request flows, and what it is built with.

> How this note was produced: [case 005](../cases/005-learn-codex-repo/case.md) · flow: [learn-codebase-flow.md](learn-codebase-flow.md) · prompt: [learn-repo-overview.md](../prompts/learn-repo-overview.md)  
> For a layer-by-layer walkthrough, see [layeredDesign.md](layeredDesign.md) (English) or [layeredDesign_cn.md](layeredDesign_cn.md) (中文).

---

## What This Project Does

**Codex CLI** is a local coding agent from OpenAI. Given a natural-language task ("fix this test", "refactor this module"), it plans and executes work with an LLM plus built-in tools — reading and editing files, running shell commands, searching the web, and calling MCP tools — all inside configurable **sandbox and approval** policies. Conversation history is persisted as **threads** and **turns**.

The shipped binary is `codex`; the npm package `@openai/codex` is a thin wrapper that execs the native binary. The same core also powers **Codex in IDE** (VS Code, Cursor, Windsurf — via app-server), the **Codex App** desktop experience (`codex app`), and the cloud agent **Codex Web** (chatgpt.com/codex).

---

## Main Architecture

A **Rust-first monorepo** ([`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs), ~130 crates) plus Node/TypeScript packaging and SDKs. Clients sit on top, capability providers (model API, execution, MCP, sandbox) below, and one `codex-core` orchestrates the middle.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Clients / Entry Points                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ codex-cli (binary)                                                          │
│      │                                                                      │
│      ├─► codex-tui           interactive terminal UI                        │
│      ├─► codex app-server    JSON-RPC (IDE, desktop, Python SDK)            │
│      └─► codex exec          headless / CI mode                             │
│                                                                             │
│ TypeScript SDK ──► codex exec JSONL    Python SDK ──► app-server stdio      │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                 Agent Core                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ codex-core                                                                  │
│      │                                                                      │
│      ├─ codex-protocol       shared Op / Event / config types               │
│      ├─ codex-tools          tool specs, routing, execution                 │
│      └─ thread-store         rollout JSONL + SQLite metadata                │
│                                                                             │
│ Codex = queue pair:  submit(Op) ──► session loop ──► receive(Event)         │
└───────────┬─────────────────┬─────────────────┬─────────────────┬───────────┘
            │                 │                 │                 │
            ▼                 ▼                 ▼                 ▼
    ┌───────────────┐ ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
    │   codex-api   │ │  exec-server  │ │   codex-mcp   │ │  sandboxing   │
    │    OpenAI     │ │  subprocess   │ │   MCP tool    │ │macOS Seatbelt │
    │   Responses   │ │     / PTY     │ │    servers    │ │  Linux bwrap  │
    │ SSE/WebSocket │ │ local/remote  │ │               │ │  Win restr.   │
    └───────────────┘ └───────────────┘ └───────────────┘ └───────────────┘
```

> The diagram uses a fixed 79-column layout so boxes and connectors align in monospace viewers.

At the center, `codex-core` exposes `Codex` as a **queue pair**: callers submit `Op`s and receive `Event`s. This one abstraction backs every surface.

```rust
pub struct Codex {
    tx_sub: Sender<Submission>,  // in:  UserInput, ExecApproval, PatchApproval, Interrupt, ...
    rx_event: Receiver<Event>,   // out: message deltas, tool progress, approval requests, turn done, ...
    // ...
}
```

`codex-core` is the central orchestrator, but the repo deliberately keeps it from growing — new features land in dedicated crates ([`codex-tools`](https://github.com/openai/codex/tree/main/codex-rs/tools), [`ext/*`](https://github.com/openai/codex/tree/main/codex-rs/ext), etc.). The next section is the map of those crates.

---

## Important Folders

### Top-level layout

| Path | Purpose |
| ---- | ------- |
| [`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs) | The Rust workspace (~130 crates) — the actual product; almost all implementation |
| [`codex-cli/`](https://github.com/openai/codex/tree/main/codex-cli) | Thin npm package (`@openai/codex`); [`bin/codex.js`](https://github.com/openai/codex/blob/main/codex-cli/bin/codex.js) just execs the platform-native Rust binary |
| [`sdk/`](https://github.com/openai/codex/tree/main/sdk) | Public **TypeScript** and **Python** SDKs for embedding Codex |
| [`docs/`](https://github.com/openai/codex/tree/main/docs) | User/contributor docs (mostly stubs pointing at developers.openai.com) |
| [`scripts/`](https://github.com/openai/codex/tree/main/scripts), [`tools/`](https://github.com/openai/codex/tree/main/tools), [`bazel/`](https://github.com/openai/codex/tree/main/bazel), [`patches/`](https://github.com/openai/codex/tree/main/patches), [`justfile`](https://github.com/openai/codex/blob/main/justfile) | Build, release, and dev tooling |
| [`third_party/`](https://github.com/openai/codex/tree/main/third_party) | Vendored dependencies (e.g. V8) |

### Key crates inside [`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs)

**Core runtime**

- [`core`](https://github.com/openai/codex/tree/main/codex-rs/core) — the heart: session state machine, turn loop, tool dispatch, history/rollout, sandbox glue
- [`protocol`](https://github.com/openai/codex/tree/main/codex-rs/protocol) — the shared vocabulary every crate speaks: `Submission`, `Op`, `Event`, `ResponseItem`, `SandboxPolicy`, `AskForApproval`
- [`tools`](https://github.com/openai/codex/tree/main/codex-rs/tools) — tool spec definitions shared between core and clients

**Front ends / entry points**

- [`cli`](https://github.com/openai/codex/tree/main/codex-rs/cli) — the `codex` multitool binary; parses subcommands and dispatches
- [`tui`](https://github.com/openai/codex/tree/main/codex-rs/tui) — interactive Ratatui terminal UI (default when no subcommand); interface design: [tui-interface-design.md](tui-interface-design.md)
- [`exec`](https://github.com/openai/codex/tree/main/codex-rs/exec) — headless `codex exec` mode (CI/scripts)
- [`mcp-server`](https://github.com/openai/codex/tree/main/codex-rs/mcp-server) — exposes Codex itself as an MCP server over stdio
- [`app-server`](https://github.com/openai/codex/tree/main/codex-rs/app-server) / [`app-server-protocol`](https://github.com/openai/codex/tree/main/codex-rs/app-server-protocol) — JSON-RPC server + v1/v2 API types backing IDE/desktop/SDK

**Model / API layer**

- [`codex-api`](https://github.com/openai/codex/tree/main/codex-rs/codex-api) — HTTP/WebSocket client for the OpenAI Responses API (SSE parsing)
- [`model-provider-info`](https://github.com/openai/codex/tree/main/codex-rs/model-provider-info) / [`model-provider`](https://github.com/openai/codex/tree/main/codex-rs/model-provider) — provider metadata + auth resolution
- [`models-manager`](https://github.com/openai/codex/tree/main/codex-rs/models-manager) — the model catalog
- [`ollama`](https://github.com/openai/codex/tree/main/codex-rs/ollama), [`lmstudio`](https://github.com/openai/codex/tree/main/codex-rs/lmstudio) — local model provider adapters

**Auth**

- [`login`](https://github.com/openai/codex/tree/main/codex-rs/login) — `AuthManager`, OAuth (ChatGPT sign-in) and API-key auth
- [`chatgpt`](https://github.com/openai/codex/tree/main/codex-rs/chatgpt) — ChatGPT-backend features (`codex apply`, connectors, cloud tasks)

**Sandboxing & safety**

- [`sandboxing`](https://github.com/openai/codex/tree/main/codex-rs/sandboxing) — cross-platform sandbox policy engine (Seatbelt / seccomp+Landlock / Windows restricted token)
- [`linux-sandbox`](https://github.com/openai/codex/tree/main/codex-rs/linux-sandbox) — Linux sandbox helper binary (Landlock + seccomp + bwrap)
- [`execpolicy`](https://github.com/openai/codex/tree/main/codex-rs/execpolicy) — Starlark rule engine that classifies command safety
- [`exec-server`](https://github.com/openai/codex/tree/main/codex-rs/exec-server) — local/remote process execution server

**Tool building blocks**

- [`apply-patch`](https://github.com/openai/codex/tree/main/codex-rs/apply-patch) — the `apply_patch` file-edit tool (also a standalone arg0 entry)
- [`codex-mcp`](https://github.com/openai/codex/tree/main/codex-rs/codex-mcp) / [`rmcp-client`](https://github.com/openai/codex/tree/main/codex-rs/rmcp-client) — Codex as an MCP **client** to external servers

**Persistence & infra**

- [`config`](https://github.com/openai/codex/tree/main/codex-rs/config), [`codex-home`](https://github.com/openai/codex/tree/main/codex-rs/codex-home) — `config.toml` loading and schema
- [`rollout`](https://github.com/openai/codex/tree/main/codex-rs/rollout) / [`rollout-trace`](https://github.com/openai/codex/tree/main/codex-rs/rollout-trace), [`thread-store`](https://github.com/openai/codex/tree/main/codex-rs/thread-store), [`state`](https://github.com/openai/codex/tree/main/codex-rs/state) — session persistence (SQLite + rollout traces)
- [`arg0`](https://github.com/openai/codex/tree/main/codex-rs/arg0) — multi-call binary dispatch (`codex` also runs as `apply_patch` / `codex-linux-sandbox`)
- [`otel`](https://github.com/openai/codex/tree/main/codex-rs/otel), [`analytics`](https://github.com/openai/codex/tree/main/codex-rs/analytics) — telemetry

### Extension crates ([`codex-rs/ext/`](https://github.com/openai/codex/tree/main/codex-rs/ext))

- [`connectors`](https://github.com/openai/codex/tree/main/codex-rs/ext/connectors) — third-party service integrations
- [`extension-api`](https://github.com/openai/codex/tree/main/codex-rs/ext/extension-api) — extension registration and hooks
- [`goal`](https://github.com/openai/codex/tree/main/codex-rs/ext/goal) — goal-tracking extension
- [`guardian`](https://github.com/openai/codex/tree/main/codex-rs/ext/guardian) — review/safety extension
- [`image-generation`](https://github.com/openai/codex/tree/main/codex-rs/ext/image-generation) — image generation tools
- [`mcp`](https://github.com/openai/codex/tree/main/codex-rs/ext/mcp) — hosted MCP server integration (contributes servers into the catalog)
- [`memories`](https://github.com/openai/codex/tree/main/codex-rs/ext/memories) — long-term memory
- [`skills`](https://github.com/openai/codex/tree/main/codex-rs/ext/skills) — agent skills
- [`web-search`](https://github.com/openai/codex/tree/main/codex-rs/ext/web-search) — web search tools

---

## How a Request Flows

Persisted context is organized as **Thread → Turn → Item**: a thread is a conversation, a turn is one user↔agent exchange, and an item is a unit of context (a message, agent reasoning, a shell command, a file edit).

Every surface runs the **same core loop** — one turn:

1. Input arrives as `Op::UserInput` on the submission queue.
2. The session assembles model context: instructions, skills, git info, memories, and the available tools.
3. `codex-api` calls the **OpenAI Responses API**; core consumes streaming `ResponseEvent`s through SSE and WebSocket-capable paths.
4. Model output streams back as `Event`s — message deltas, reasoning, tool calls.
5. On a tool call, `ToolRouter` dispatches to a handler (shell, `apply_patch`, file search, MCP, …); commands run through the sandbox, **pausing for approval** when policy requires.
6. Tool results feed back into the model; steps 3–6 repeat until the turn completes.
7. Events render in the client; history persists to the thread store.

**The three surfaces differ only in how input arrives and events are rendered:**

- **TUI** (`codex`) — [`codex-tui`](https://github.com/openai/codex/tree/main/codex-rs/tui) uses the app-server client with an embedded in-process app-server by default, so it keeps app-server semantics without requiring a separate process.
- **app-server** (`codex app-server`) — the same loop over **JSON-RPC 2.0** (stdio/JSONL, unix socket, or websocket) for IDEs, desktop, and the Python SDK.
- **Headless** (`codex exec`) — submit a prompt, run the loop, print structured output, exit; the TypeScript SDK wraps this mode over stdin/stdout JSONL.

app-server message exchange:

```text
Client                    app-server                  codex-core
  │── initialize ──────────►│                           │
  │◄── initialized ─────────│                           │
  │── thread/start ────────►│── spawn Codex session ───►│
  │── turn/start ──────────►│── Op::UserInput ─────────►│
  │◄── item/started ────────│◄── Event stream ──────────│
  │◄── item/agentMessage/delta                          │
  │◄── turn/completed ──────│◄── turn done ─────────────│
```

### Sandboxing in the execution path

Tool execution is wrapped by platform-specific isolation, governed by `SandboxPolicy` plus approval gates:

| Platform    | Mechanism                                                   |
| ----------- | ----------------------------------------------------------- |
| **macOS**   | Seatbelt (`/usr/bin/sandbox-exec`)                          |
| **Linux**   | bubblewrap (`bwrap`) + Landlock/seccomp                     |
| **Windows** | Restricted-token sandbox (elevated / unelevated backends)   |

Before the OS sandbox even applies, [`execpolicy`](https://github.com/openai/codex/tree/main/codex-rs/execpolicy) (a Starlark rule engine) classifies whether a command is safe or needs approval. `exec-server` can run locally or register with a remote registry (Noise-encrypted relay over WebSocket) for cross-machine execution.

### Key code anchors

| Concept | Location |
| ------- | -------- |
| `Codex` handle (queue pair) | [`core/src/session/mod.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/session/mod.rs) |
| Submission loop (`Op` dispatch) | [`core/src/session/handlers.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/session/handlers.rs) |
| Turn execution (`run_turn`) | [`core/src/session/turn.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/session/turn.rs) |
| Model client (SSE / WebSocket) | [`core/src/client.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/client.rs) |
| Tool routing / dispatch | [`core/src/tools/router.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/tools/router.rs) |
| Shell exec + sandbox selection | [`core/src/exec.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/exec.rs) |
| Multi-call binary dispatch (`arg0`) | [`arg0/src/lib.rs`](https://github.com/openai/codex/tree/main/codex-rs/arg0/src/lib.rs) |

---

## MCP: Both Directions

Codex speaks the **Model Context Protocol on both sides**.

- **As a server** ([`mcp-server`](https://github.com/openai/codex/tree/main/codex-rs/mcp-server)) — a stdio JSON-RPC loop exposing exactly two tools, `codex` and `codex-reply`. A `tools/call` spins up a full Codex session, submits `Op::UserInput`, and streams core `EventMsg`s back as MCP responses; approval requests become MCP **elicitations**. Any other agent can thus drive a complete Codex session.
- **As a client** — to gain extra tools, Codex connects out to external MCP servers. [`rmcp-client`](https://github.com/openai/codex/tree/main/codex-rs/rmcp-client) (`RmcpClient`, stdio/HTTP + OAuth) is the transport; [`codex-mcp`](https://github.com/openai/codex/tree/main/codex-rs/codex-mcp) (`McpConnectionManager`) starts one client per server, aggregates their tools, and handles name collisions. During a turn, those tools register as `McpHandler`s and calls route `handle_mcp_tool_call` → `McpConnectionManager::call_tool` → `RmcpClient`. [`ext/mcp`](https://github.com/openai/codex/tree/main/codex-rs/ext/mcp) contributes hosted servers (e.g. Codex Apps) via feature flags.

---

## Technologies Used

| Category               | Stack                                                              |
| ---------------------- | ------------------------------------------------------------------ |
| **Language**           | Rust (primary), TypeScript (SDKs, npm wrapper), Python (SDK)       |
| **Async runtime**      | Tokio                                                              |
| **Terminal UI**        | ratatui + crossterm                                                |
| **LLM API**            | OpenAI Responses API (SSE and WebSocket streaming paths)            |
| **Protocols**          | JSON-RPC 2.0, JSONL, MCP (Model Context Protocol)                  |
| **Persistence**        | rollout JSONL history; SQLite state DB for queryable metadata       |
| **Sandboxing**         | macOS Seatbelt, Linux bubblewrap, Windows restricted-token sandbox |
| **Remote execution**   | WebSocket + Noise-encrypted relay (exec-server remote mode)        |
| **Build**              | Cargo workspace + Bazel; `just` for dev commands                   |
| **Package management** | pnpm (Node), Cargo (Rust)                                          |
| **Testing**            | `insta` snapshot tests (TUI), integration tests in `core/tests/suite`    |
| **Observability**      | OpenTelemetry (`codex-otel`), structured tracing                   |
| **Schema**             | JSON Schema for config; TypeScript generation for app-server API   |

---

## Mental Model

Four cooperating systems, all composed around the core session/turn loop:

1. **Conversation engine** — threads, turns, context assembly, model streaming
2. **Tool runtime** — shell, patches, search, MCP, extensions
3. **Safety layer** — sandboxing, exec policy, approvals
4. **Integration surface** — TUI, app-server JSON-RPC, SDKs, `exec` mode

The `codex` binary is the single entry point; everything else follows one rhythm: **stream model responses → execute tools → persist state → emit events.**

---

## Further Reading

### This repo (codex-labs)

- [layeredDesign.md](layeredDesign.md) / [layeredDesign_cn.md](layeredDesign_cn.md) — layer-by-layer architecture walkthrough
- [resources/links.md](../resources/links.md) / [links_zh.md](../resources/links_zh.md) — curated external links

### [openai/codex](https://github.com/openai/codex) on GitHub

- [README.md](https://github.com/openai/codex/blob/main/README.md) — project overview and quickstart
- [AGENTS.md](https://github.com/openai/codex/blob/main/AGENTS.md) — contributor conventions and crate guidance
- [docs/contributing.md](https://github.com/openai/codex/blob/main/docs/contributing.md) — how to contribute
- [docs/install.md](https://github.com/openai/codex/blob/main/docs/install.md) — install and build
- [codex-rs/app-server/README.md](https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md) — app-server protocol and API reference
- [codex-rs/core/README.md](https://github.com/openai/codex/blob/main/codex-rs/core/README.md) — sandbox platform requirements
- [codex-rs/codex-api/README.md](https://github.com/openai/codex/blob/main/codex-rs/codex-api/README.md) — Responses API client interface
- [codex-rs/exec-server/README.md](https://github.com/openai/codex/blob/main/codex-rs/exec-server/README.md) — process execution server
- [codex-rs/protocol/README.md](https://github.com/openai/codex/blob/main/codex-rs/protocol/README.md) — shared protocol types
- [codex-rs/tools/README.md](https://github.com/openai/codex/blob/main/codex-rs/tools/README.md) — tool specs and contracts

### Official docs

- [Codex documentation](https://developers.openai.com/codex)

---

_Generated from exploration of the [openai/codex](https://github.com/openai/codex) repository._
