# Codex Layered Architecture

**English** | [中文](layered-design_zh.md)

> **Answers:** how the crates layer, and what each layer is responsible for.
> **Read first:** [architecture.md](architecture.md).
> **Verified against:** [openai/codex](https://github.com/openai/codex)@`da4c8ca` (2026-07-03) — re-check `git diff da4c8ca..HEAD -- codex-rs/` before trusting details.

A layer-by-layer walkthrough of the Codex repository. For the crate-by-crate map and request flow, see [architecture.md](architecture.md).

---

## Overview: How the Layers Connect

Think of the stack as: **entry points on top, protocol and brain in the middle, capabilities and safety at the bottom**.

```text
You (user / IDE / SDK)
        │
        ▼
  ① Entry points / ⑧ IDE integration   ← how you get in
        │
        ▼
  ② Protocol                            ← shared vocabulary
        │
        ▼
  ③ Core                                ← how the agent thinks and runs
        │
   ┌────┴────┬──────────┬──────────┐
   ▼         ▼          ▼          ▼
⑦ Extensions  ④ API    ⑤ Execution  ⑥ Sandboxing
```

---

## ① Entry points — `codex-cli`

**Role:** The **single executable entry point** for the whole system (the `codex` command).

When you run `codex` in a terminal, you are running the binary built from [`codex-rs/cli`](https://github.com/openai/codex/tree/main/codex-rs/cli). It does **not** do agent reasoning itself — it **routes** to the right mode:

| Subcommand / mode | What it does |
|-------------------|--------------|
| Default (no args) | Launches the **TUI** interactive terminal UI |
| `codex exec` | **Headless mode** for scripts and CI |
| `codex app-server` | Starts the **JSON-RPC server** for IDEs and SDKs |
| `codex mcp` / `plugin` / `login` etc. | Config, auth, and plugin management |

The npm package `@openai/codex` is a thin **wrapper** that ultimately invokes this native binary.

**Why this layer matters:** One command unifies how you use Codex; every mode shares the same core underneath.

---

## ② Protocol — `codex-protocol`

**Role:** The **shared language** between components — types only, almost no business logic.

This crate holds protocol types used by core, TUI, and app-server, with minimal dependencies.

Key concepts:

| Type | Direction | Meaning |
|------|-----------|---------|
| `Op` | Caller → Core | What you want Core to **do** (user input, approvals, interrupt, etc.) |
| `Event` / `EventMsg` | Core → Caller | What Core **reports** (streaming output, tool progress, approval requests) |
| `ThreadId`, `TurnItem`, `Settings`, etc. | Both | Shared definitions for sessions, config, and persistence |

For example, `Op::UserInput` means "start a turn with this input"; `EventMsg` carries message deltas, tool calls, turn completion, and more.

**Why this layer matters:** TUI, app-server, and exec do not each invent their own message format. They all talk to core via `Op` / `Event`. You can change a UI without rewriting core internals, as long as the protocol stays compatible.

---

## ③ Core — `codex-core`

**Role:** The **agent brain and orchestrator** — sessions, context, model calls, and tool routing all live here.

The central abstraction is the **queue pair**:

```text
Caller                    codex-core
   │                          │
   │── submit(Op) ───────────►│  submission queue
   │                          │  session loop
   │◄── Event stream ─────────│  event queue
```

A single **turn** in core roughly follows these steps:

1. Receive `Op::UserInput`
2. **Build context**: system instructions, skills, plugins, git info, memories, MCP tools, etc.
3. Call the **OpenAI Responses API** via `codex-api`; consume streaming `ResponseEvent`s through SSE and WebSocket-capable paths
4. Parse **tool calls** from the stream and hand them to `ToolRouter`
5. When needed, emit **approval requests** (shell, patches, network) and wait for `Op::ExecApproval`, etc.
6. Feed tool results back to the model; loop until the turn completes
7. Push `Event`s outward; persist history to **thread-store**

**Why this layer matters:** All rules for "how the agent works" are centralized here. Upper layers only need to send `Op` and receive `Event` — they do not need to know how models or tools are wired.

> The repo deliberately **limits core growth**: new features tend to land in `codex-tools`, `ext/*`, and other crates; core orchestrates.

---

## ④ API client — `codex-api` + `codex-client`

**Role:** The **network layer** to the **OpenAI Responses API** — request models, SSE parsing, WebSocket-capable streaming, retries, auth headers.

Division of labor:

| Crate | Responsibility |
|-------|----------------|
| `codex-client` | Generic HTTP transport |
| `codex-api` | Request bodies for Responses / Compact / Memory APIs, SSE parsing, `ResponseEvent` stream types |

Core assembles `instructions`, `input`, and `tools`, then hands off to this layer. What comes back are structured `ResponseEvent`s (text deltas, function calls, completion signals, etc.).

**Why this layer matters:** "How we talk to OpenAI" is separated from business logic. Endpoint changes, retries, SSE parsing, and streaming transport details mostly touch this layer and the core model client, not session/turn code.

---

## ⑤ Execution — core exec path + `exec-server`

**Role:** Where processes **actually run on the machine** — shells, subprocess lifecycle, stdin/stdout.

By default this happens **inside `codex-core`**: `core/src/exec.rs` (`process_exec_tool_call` → `spawn_child_async`) spawns the process directly, wrapped by sandboxing (⑥). Core decides "run `cargo test`"; the core exec path spawns it and streams the output back.

[`exec-server`](https://github.com/openai/codex/tree/main/codex-rs/exec-server) is an **experimental** extraction of that execution into a standalone service, for two scenarios:

- Local: `codex exec-server` (marked `[EXPERIMENTAL]`) over WebSocket JSON-RPC
- Remote: registers with an environment registry and controls processes across machines via **Noise-encrypted relay** (app-server and exec-server can run on different OSes)

**Why this layer matters:** Process spawning is a distinct concern with its own sandbox wrapping — and factoring it into `exec-server` lets the same execution run locally or across machines, hardened independently.

---

## ⑥ Sandboxing — `sandboxing` and friends

**Role:** The **safety boundary** — limits what filesystem and network access is allowed.

| Platform | Mechanism |
|----------|-----------|
| macOS | Seatbelt (`sandbox-exec`) |
| Linux | bubblewrap (`bwrap`) + Landlock/seccomp |
| Windows | Restricted-token sandbox |

Policies come from `SandboxPolicy` / permissions config (read-only, workspace-write, network rules, etc.). Before the core exec path starts a command, sandboxing wraps it according to policy.

**Why this layer matters:** No matter how capable the model is, there is still **platform-level isolation** plus **user approvals** before touching disk or network.

---

## ⑦ Extensions — `ext/*`

**Role:** **Pluggable capabilities** so core does not absorb everything.

| Extension | Capability |
|-----------|------------|
| `ext/skills` | Agent skills |
| `ext/mcp` | MCP tool servers |
| `ext/connectors` | Third-party service integrations |
| `ext/memories` | Long-term memory |
| `ext/web-search` | Web search tools |
| `ext/guardian` | Review / safety |
| `ext/goal` | Goal tracking |

Core **pulls these in** when building context and registering tools, but implementation stays in each crate.

**Why this layer matters:** New capabilities can be developed, tested, and toggled independently — aligned with the repo's "keep core lean" direction.

---

## ⑧ IDE integration — `app-server` + `app-server-protocol`

**Role:** The **external API layer** for rich clients (VS Code extension, desktop app, Python SDK).

Unlike the TUI, this layer does not render a terminal. Instead:

- Outward: **JSON-RPC 2.0** (stdio JSONL, unix socket, etc.)
- Inward: still `Op` / `Event`, driving `codex-core`

External concepts (more product-shaped than internal `Op`):

```text
Thread (conversation) → Turn (one exchange) → Item (message / command / edit / reasoning)
```

Typical flow:

```text
initialize → thread/start → turn/start → receive item/* notifications → turn/completed
```

`app-server-protocol` defines v1/v2 API types and can generate TypeScript schemas so IDEs stay aligned with CLI versions.

**Why this layer matters:** Core is wrapped as a **stable, versioned RPC surface** so editors and app-server clients do not need to embed the Rust TUI. The TypeScript SDK is the exception: it wraps `codex exec --experimental-json` over stdin/stdout JSONL instead of app-server.

---

## End-to-end: One Request Through the Stack

Example: "Fix this test" from VS Code:

```text
1. VS Code extension
      │  JSON-RPC: turn/start
      ▼
2. app-server (⑧)
      │  converts to Op::UserInput
      ▼
3. codex-protocol (②)  type alignment
      ▼
4. codex-core (③)
      │  builds context; loads ext/skills, mcp (⑦)
      │  calls codex-api (④) → OpenAI
      │  model requests a test run → ToolRouter
      ▼
5. core exec path (⑤)  spawns shell (core/src/exec.rs)
      │  wrapped by sandboxing (⑥)
      ▼
6. results return to core → fed back to model → Event stream
      ▼
7. app-server → item/completed etc. → VS Code UI
```

---

## Cheat sheet: One line per layer

| Layer | In one sentence |
|-------|-----------------|
| Entry points | The `codex` command; picks TUI, exec, or app-server |
| Protocol | Shared `Op` / `Event` / config types |
| Core | Agent main loop: context, model, tools, persistence |
| API client | Talks to the OpenAI Responses API |
| Execution | Runs subprocesses — in core by default; `exec-server` (experimental) for local/remote |
| Sandboxing | Restricts filesystem and network access |
| Extensions | Pluggable skills, MCP, memory, and more |
| IDE integration | JSON-RPC facade for editors and SDKs |

---

## Further reading

### This repo (codex-labs)

- [architecture.md](architecture.md) / [architecture_zh.md](architecture_zh.md) — repository architecture overview
- [resources/links.md](../../resources/links.md) / [links_zh.md](../../resources/links_zh.md) — curated external links

### [openai/codex](https://github.com/openai/codex) on GitHub

- [README.md](https://github.com/openai/codex/blob/main/README.md) — project overview and quickstart
- [AGENTS.md](https://github.com/openai/codex/blob/main/AGENTS.md) — contributor conventions and crate guidance
- [codex-rs/app-server/README.md](https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md) — app-server protocol and API reference
- [codex-rs/core/README.md](https://github.com/openai/codex/blob/main/codex-rs/core/README.md) — sandbox platform requirements
- [codex-rs/codex-api/README.md](https://github.com/openai/codex/blob/main/codex-rs/codex-api/README.md) — Responses API client interface
- [codex-rs/exec-server/README.md](https://github.com/openai/codex/blob/main/codex-rs/exec-server/README.md) — process execution server
- [codex-rs/protocol/README.md](https://github.com/openai/codex/blob/main/codex-rs/protocol/README.md) — shared protocol types
- [codex-rs/tools/README.md](https://github.com/openai/codex/blob/main/codex-rs/tools/README.md) — tool specs and contracts

### Official docs

- [Codex documentation](https://developers.openai.com/codex)
