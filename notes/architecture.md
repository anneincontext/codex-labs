# Codex Repository Architecture

**English** | [中文](architecture_cn.md)

High-level overview of the [openai/codex](https://github.com/openai/codex) repository — what it does, how it is structured, how requests flow, and what technologies it uses.

> For a layer-by-layer walkthrough, see [layeredDesign.md](layeredDesign.md) (English) or [layeredDesign_cn.md](layeredDesign_cn.md) (中文).

---

## What This Project Does

**Codex CLI** is a local coding agent from OpenAI. It runs on your machine and can:

- Accept natural-language tasks ("fix this test", "refactor this module", etc.)
- Plan and execute work using LLM reasoning plus built-in tools
- Read and edit files, run shell commands, search the web, and call MCP tools
- Operate inside configurable sandbox and approval policies
- Persist conversation history as **threads** and **turns**

The shipped binary is `codex`. The npm package `@openai/codex` is a thin wrapper that invokes the native binary.

Related products (not all in this repo):

- **Codex in IDE** — VS Code, Cursor, Windsurf via the app-server protocol
- **Codex App** — desktop experience (`codex app`)
- **Codex Web** — cloud-based agent at chatgpt.com/codex

---

## Main Architecture

The repo is a **Rust-first monorepo** ([`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs)) with ~100 crates, plus Node/TypeScript packaging and SDKs.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Clients / Entry Points                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ codex-cli (binary)                                                          │
│      │                                                                      │
│      ├─► codex-tui           interactive terminal UI                        │
│      ├─► codex app-server    JSON-RPC (IDE, desktop, SDKs)                  │
│      └─► codex exec          headless / CI mode                             │
│                                                                             │
│ SDKs (TypeScript, Python) ─────────────────────────────► app-server         │
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
│      └─ thread-store         SQLite persistence + rollout traces            │
│                                                                             │
│ Codex = queue pair:  submit(Op) ──► session loop ──► receive(Event)         │
└───────────┬─────────────────┬─────────────────┬─────────────────┬───────────┘
            │                 │                 │                 │
            ▼                 ▼                 ▼                 ▼
    ┌───────────────┐ ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
    │   codex-api   │ │  exec-server  │ │   codex-mcp   │ │  sandboxing   │
    │    OpenAI     │ │  subprocess   │ │   MCP tool    │ │macOS Seatbelt │
    │   Responses   │ │     / PTY     │ │    servers    │ │  Linux bwrap  │
    │   API (SSE)   │ │ local/remote  │ │               │ │  Win restr.   │
    │               │ │               │ │               │ │               │
    │               │ │               │ │               │ │               │
    │               │ │               │ │               │ │               │
    └───────────────┘ └───────────────┘ └───────────────┘ └───────────────┘
```

> **Note:** This ASCII diagram uses a fixed 79-column layout so boxes, connectors, and bottom modules align in monospace viewers.

### Layered design

| Layer | Crate(s) | Role |
| ----- | -------- | ---- |
| **Entry points** | [`codex-cli`](https://github.com/openai/codex/tree/main/codex-rs/cli) | `codex` binary dispatches to TUI, `app-server`, `exec`, MCP, plugins, etc. |
| **Protocol** | [`codex-protocol`](https://github.com/openai/codex/tree/main/codex-rs/protocol) | Shared types for ops, events, config, thread/turn items |
| **Core** | [`codex-core`](https://github.com/openai/codex/tree/main/codex-rs/core) | Agent loop, session management, context building, tool orchestration |
| **API client** | [`codex-api`](https://github.com/openai/codex/tree/main/codex-rs/codex-api), [`codex-client`](https://github.com/openai/codex/tree/main/codex-rs/codex-client) | Wire protocol to OpenAI Responses API (SSE streaming) |
| **Execution** | [`exec-server`](https://github.com/openai/codex/tree/main/codex-rs/exec-server) | Spawns and controls subprocesses; can run locally or remotely |
| **Sandboxing** | [`sandboxing`](https://github.com/openai/codex/tree/main/codex-rs/sandboxing), [`linux-sandbox`](https://github.com/openai/codex/tree/main/codex-rs/linux-sandbox), [`windows-sandbox-rs`](https://github.com/openai/codex/tree/main/codex-rs/windows-sandbox-rs) | Platform-specific isolation |
| **Extensions** | [`ext/*`](https://github.com/openai/codex/tree/main/codex-rs/ext) | Skills, MCP, connectors, memories, web search, goals, etc. |
| **IDE integration** | [`app-server`](https://github.com/openai/codex/tree/main/codex-rs/app-server), [`app-server-protocol`](https://github.com/openai/codex/tree/main/codex-rs/app-server-protocol) | JSON-RPC API for VS Code extension and other rich UIs |

[`codex-core`](https://github.com/openai/codex/tree/main/codex-rs/core) is the central orchestrator, but the repo actively resists growing it further. New features tend to land in dedicated crates ([`codex-tools`](https://github.com/openai/codex/tree/main/codex-rs/tools), [`ext/*`](https://github.com/openai/codex/tree/main/codex-rs/ext), etc.).

### Core abstraction: queue pair

`codex-core` exposes `Codex` as a high-level handle that operates as a **queue pair**: callers send `Op` submissions and receive `Event` streams.

```rust
pub struct Codex {
    pub(crate) tx_sub: Sender<Submission>,
    pub(crate) rx_event: Receiver<Event>,
    pub(crate) agent_status: watch::Receiver<AgentStatus>,
    pub(crate) session: Arc<Session>,
    // ...
}
```

Key inbound operations (`Op`): `UserInput`, `ExecApproval`, `PatchApproval`, `Interrupt`, `ThreadSettings`, etc.

Key outbound messages (`Event` / `EventMsg`): agent message deltas, tool progress, approval requests, turn completion, errors.

---

## Important Folders

| Path | Purpose |
| ---- | ------- |
| [`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs) | Main Rust workspace — almost all implementation |
| [`codex-rs/core/`](https://github.com/openai/codex/tree/main/codex-rs/core) | Agent business logic: sessions, turns, tools, context, approvals |
| [`codex-rs/cli/`](https://github.com/openai/codex/tree/main/codex-rs/cli) | `codex` binary entry point and subcommand routing |
| [`codex-rs/tui/`](https://github.com/openai/codex/tree/main/codex-rs/tui) | Terminal UI (ratatui) |
| [`codex-rs/app-server/`](https://github.com/openai/codex/tree/main/codex-rs/app-server) | JSON-RPC server for IDE/desktop integrations |
| [`codex-rs/app-server-protocol/`](https://github.com/openai/codex/tree/main/codex-rs/app-server-protocol) | v1/v2 API types (Rust + generated TypeScript schemas) |
| [`codex-rs/protocol/`](https://github.com/openai/codex/tree/main/codex-rs/protocol) | Internal protocol types shared across core, TUI, app-server |
| [`codex-rs/codex-api/`](https://github.com/openai/codex/tree/main/codex-rs/codex-api) | OpenAI Responses API client and SSE parsing |
| [`codex-rs/exec-server/`](https://github.com/openai/codex/tree/main/codex-rs/exec-server) | Remote/local process execution server |
| [`codex-rs/sandboxing/`](https://github.com/openai/codex/tree/main/codex-rs/sandboxing) | Cross-platform sandbox policies and enforcement |
| [`codex-rs/tools/`](https://github.com/openai/codex/tree/main/codex-rs/tools) | Tool specs, adapters, and shared tool contracts |
| [`codex-rs/ext/`](https://github.com/openai/codex/tree/main/codex-rs/ext) | Optional extensions: skills, MCP, connectors, memories, web-search, etc. |
| [`codex-rs/config/`](https://github.com/openai/codex/tree/main/codex-rs/config) | `config.toml` loading and schema |
| [`codex-rs/thread-store/`](https://github.com/openai/codex/tree/main/codex-rs/thread-store) | Persistent thread/turn storage (SQLite) |
| [`codex-rs/rollout/`](https://github.com/openai/codex/tree/main/codex-rs/rollout) | Session rollout traces for debugging/replay |
| [`codex-cli/`](https://github.com/openai/codex/tree/main/codex-cli) | npm package (`@openai/codex`) — distribution wrapper |
| [`sdk/`](https://github.com/openai/codex/tree/main/sdk) | TypeScript and Python SDKs for embedding Codex |
| [`docs/`](https://github.com/openai/codex/tree/main/docs) | Contributor docs (install, config, sandbox, contributing) |
| [`scripts/`](https://github.com/openai/codex/tree/main/scripts), [`bazel/`](https://github.com/openai/codex/tree/main/bazel), [`justfile`](https://github.com/openai/codex/blob/main/justfile) | Build, test, and dev automation |
| [`third_party/`](https://github.com/openai/codex/tree/main/third_party) | Vendored dependencies (e.g. V8) |

### Extension crates ([`codex-rs/ext/`](https://github.com/openai/codex/tree/main/codex-rs/ext))

- [`connectors`](https://github.com/openai/codex/tree/main/codex-rs/ext/connectors) — third-party service integrations
- [`extension-api`](https://github.com/openai/codex/tree/main/codex-rs/ext/extension-api) — extension registration and hooks
- [`goal`](https://github.com/openai/codex/tree/main/codex-rs/ext/goal) — goal-tracking extension
- [`guardian`](https://github.com/openai/codex/tree/main/codex-rs/ext/guardian) — review/safety extension
- [`image-generation`](https://github.com/openai/codex/tree/main/codex-rs/ext/image-generation) — image generation tools
- [`mcp`](https://github.com/openai/codex/tree/main/codex-rs/ext/mcp) — MCP server integration
- [`memories`](https://github.com/openai/codex/tree/main/codex-rs/ext/memories) — long-term memory
- [`skills`](https://github.com/openai/codex/tree/main/codex-rs/ext/skills) — agent skills
- [`web-search`](https://github.com/openai/codex/tree/main/codex-rs/ext/web-search) — web search tools

---

## How Requests Flow Through the System

### Core primitives (app-server model)

The API exposes three top-level concepts:

- **Thread** — a conversation between user and agent; contains multiple turns
- **Turn** — one exchange, typically user input → agent response; contains multiple items
- **Item** — a persisted unit of context: user message, agent reasoning, shell command, file edit, etc.

### Interactive TUI (`codex`)

1. User types a prompt in the terminal UI (`codex-tui`).
2. TUI sends a **turn start** with user input (via app-server or core directly).
3. Core receives `Op::UserInput` on an async submission queue.
4. A **session** builds model context: system instructions, skills, plugins, git info, memories, MCP tools, etc.
5. Core calls the **OpenAI Responses API** via `codex-api`, streaming SSE `ResponseEvent`s.
6. As the model streams output, core emits **events** (message deltas, reasoning, tool calls, etc.).
7. When the model requests a **tool call** (shell, `apply_patch`, file search, MCP, etc.):
   - `ToolRouter` routes to the right handler
   - **Approvals** may pause for user consent (exec commands, patches, network)
   - Commands run through **exec-server** under **sandbox policy**
8. Tool results are fed back into the model; the loop continues until the turn completes.
9. Events stream to the UI; history is persisted to the **thread store**.

### IDE / SDK (`codex app-server`)

Same core loop, but over **JSON-RPC 2.0** on stdio (JSONL), unix socket, or websocket:

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

Lifecycle:

1. **Initialize** — client sends `initialize` with `clientInfo`; server responds with capabilities
2. **Start/resume thread** — `thread/start`, `thread/resume`, or `thread/fork`
3. **Begin turn** — `turn/start` with `threadId` and user input
4. **Stream events** — notifications: `item/started`, `item/completed`, `item/agentMessage/delta`, tool progress
5. **Finish turn** — `turn/completed` with final state and token usage

### Headless (`codex exec`)

Non-interactive mode for CI/scripts: submits a prompt, runs the agent loop, prints structured output, exits.

### Sandboxing in the execution path

Platform-specific enforcement wraps tool execution:

| Platform    | Mechanism                                                        |
| ----------- | ---------------------------------------------------------------- |
| **macOS**   | Seatbelt (`/usr/bin/sandbox-exec`)                               |
| **Linux**   | bubblewrap (`bwrap`), with Landlock fallback for legacy policies |
| **Windows** | Restricted-token sandbox (elevated and unelevated backends)      |

Exec commands and filesystem access are governed by `SandboxPolicy` / permissions profiles. User approval gates can block or allow individual operations.

### Remote execution

`exec-server` can run locally (WebSocket) or register with a remote environment registry. Remote mode uses Noise-encrypted relay frames over WebSocket for cross-machine process control — useful when the app-server and exec-server run on different OSes.

---

## Technologies Used

| Category               | Stack                                                              |
| ---------------------- | ------------------------------------------------------------------ |
| **Language**           | Rust (primary), TypeScript (SDKs, npm wrapper), Python (SDK)       |
| **Async runtime**      | Tokio                                                              |
| **Terminal UI**        | ratatui + crossterm                                                |
| **LLM API**            | OpenAI Responses API (SSE streaming)                               |
| **Protocols**          | JSON-RPC 2.0, JSONL, MCP (Model Context Protocol)                  |
| **Persistence**        | SQLite (threads), rollout trace files                              |
| **Sandboxing**         | macOS Seatbelt, Linux bubblewrap, Windows restricted-token sandbox |
| **Remote execution**   | WebSocket + Noise-encrypted relay (exec-server remote mode)        |
| **Build**              | Cargo workspace + Bazel; `just` for dev commands                   |
| **Package management** | pnpm (Node), Cargo (Rust)                                          |
| **Testing**            | `insta` snapshot tests (TUI), integration tests in `core/suite`    |
| **Observability**      | OpenTelemetry (`codex-otel`), structured tracing                   |
| **Schema**             | JSON Schema for config; TypeScript generation for app-server API   |

---

## Mental Model

Think of Codex as four cooperating systems:

1. **Conversation engine** — threads, turns, context assembly, model streaming
2. **Tool runtime** — shell, patches, search, MCP, extensions
3. **Safety layer** — sandboxing, exec policy, approvals
4. **Integration surface** — TUI, app-server JSON-RPC, SDKs, `exec` mode

The `codex` binary is the single entry point. Everything else composes around the core session/turn loop: stream model responses → execute tools → persist state → emit events.

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
