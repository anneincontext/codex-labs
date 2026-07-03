# Codex Layered Architecture

**English** | [中文](layeredDesign_cn.md)

A layer-by-layer walkthrough of the Codex repository, based on the **Layered design** section in [architecture.md](architecture.md).

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
3. Call the **OpenAI Responses API** via `codex-api`, consuming SSE `ResponseEvent`s
4. Parse **tool calls** from the stream and hand them to `ToolRouter`
5. When needed, emit **approval requests** (shell, patches, network) and wait for `Op::ExecApproval`, etc.
6. Feed tool results back to the model; loop until the turn completes
7. Push `Event`s outward; persist history to **thread-store**

**Why this layer matters:** All rules for "how the agent works" are centralized here. Upper layers only need to send `Op` and receive `Event` — they do not need to know how models or tools are wired.

> The repo deliberately **limits core growth**: new features tend to land in `codex-tools`, `ext/*`, and other crates; core orchestrates.

---

## ④ API client — `codex-api` + `codex-client`

**Role:** The **network layer** to the **OpenAI Responses API** — requests, SSE streaming, retries, auth headers.

Division of labor:

| Crate | Responsibility |
|-------|----------------|
| `codex-client` | Generic HTTP transport |
| `codex-api` | Request bodies for Responses / Compact / Memory APIs, SSE parsing, `ResponseEvent` streams |

Core assembles `instructions`, `input`, and `tools`, then hands off to this layer. What comes back are structured `ResponseEvent`s (text deltas, function calls, completion signals, etc.).

**Why this layer matters:** "How we talk to OpenAI" is separated from business logic. Endpoint changes, retries, and SSE parsing mostly touch this layer, not session/turn code.

---

## ⑤ Execution — `exec-server`

**Role:** Where processes **actually run on the machine** — shells, PTYs, subprocess lifecycle.

- Local: `codex exec-server` over WebSocket JSON-RPC
- Remote: registers with an environment registry and controls processes across machines via **Noise-encrypted relay** (app-server and exec-server can run on different OSes)

Core decides "run `cargo test`"; **exec-server** spawns the process, handles stdin/stdout, and manages background terminals.

**Why this layer matters:** Dangerous operations (spawning processes) are pulled out of core for local/remote deployment and independent hardening.

---

## ⑥ Sandboxing — `sandboxing` and friends

**Role:** The **safety boundary** — limits what filesystem and network access is allowed.

| Platform | Mechanism |
|----------|-----------|
| macOS | Seatbelt (`sandbox-exec`) |
| Linux | bubblewrap (`bwrap`), with Landlock fallback in some cases |
| Windows | Restricted-token sandbox |

Policies come from `SandboxPolicy` / permissions config (read-only, workspace-write, network rules, etc.). Before exec-server starts a command, sandboxing wraps it according to policy.

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

**Role:** The **external API layer** for rich clients (VS Code extension, desktop app, TypeScript/Python SDKs).

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

`app-server-protocol` defines v2 API types and can generate TypeScript schemas so IDEs stay aligned with CLI versions.

**Why this layer matters:** Core is wrapped as a **stable, versioned RPC surface** so editors and SDKs do not need to embed the Rust TUI.

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
5. exec-server (⑤)  spawns shell
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
| Execution | Actually runs subprocesses locally or remotely |
| Sandboxing | Restricts filesystem and network access |
| Extensions | Pluggable skills, MCP, memory, and more |
| IDE integration | JSON-RPC facade for editors and SDKs |

---

## Further reading

### This repo (codex-labs)

- [architecture.md](architecture.md) / [architecture_cn.md](architecture_cn.md) — repository architecture overview
- [resources/links.md](../resources/links.md) / [links_zh.md](../resources/links_zh.md) — curated external links

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