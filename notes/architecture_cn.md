# Codex 仓库架构总览

[English](architecture.md) | **中文**

[openai/codex](https://github.com/openai/codex) 仓库的高层概览——项目做什么、如何组织、请求如何流转、用了哪些技术。

> 这份笔记怎么来的：[案例 005](../cases/005-learn-codex-repo/case_zh.md) · 链路：[learn-codebase-flow_cn.md](learn-codebase-flow_cn.md) · prompt：[learn-repo-overview_zh.md](../prompts/learn-repo-overview_zh.md)  
> 逐层详解见 [layeredDesign_cn.md](layeredDesign_cn.md)（中文）或 [layeredDesign.md](layeredDesign.md)（English）。

---

## 项目做什么

**Codex CLI** 是 OpenAI 的本地编程代理。给它一个自然语言任务（「修这个测试」「重构这个模块」），它就用 LLM 加内置工具来规划和执行——读写文件、跑 shell 命令、搜索网页、调用 MCP 工具——全程都在可配置的**沙箱和审批**策略下进行。对话历史持久化为 **thread（会话）** 和 **turn（轮次）**。

发布的二进制叫 `codex`；npm 包 `@openai/codex` 只是调用原生二进制的薄包装。同一套 core 也驱动 **Codex in IDE**（VS Code、Cursor、Windsurf——经 app-server）、**Codex App** 桌面体验（`codex app`），以及云端代理 **Codex Web**（chatgpt.com/codex）。

---

## 整体架构

一个 **Rust 优先的 monorepo**（[`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs)，约 130 个 crate），外加 Node/TypeScript 打包和 SDK。客户端在上，能力提供方（模型 API、执行、MCP、沙箱）在下，中间由一个 `codex-core` 编排。

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                                客户端 / 入口                                  │
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
│                                   代理核心                                   │
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
    └───────────────┘ └───────────────┘ └───────────────┘ └───────────────┘
```

> 图中采用固定 79 列布局，让方框和连线在等宽字体下对齐；crate 名保留英文。

核心处，`codex-core` 把 `Codex` 暴露为一个 **队列对**：调用方发送 `Op`，接收 `Event` 流。这一个抽象支撑了所有接入面。

```rust
pub struct Codex {
    tx_sub: Sender<Submission>,  // 入站：UserInput、ExecApproval、PatchApproval、Interrupt……
    rx_event: Receiver<Event>,   // 出站：消息增量、工具进度、审批请求、turn 完成……
    // ...
}
```

`codex-core` 是中心编排器，但仓库有意控制它的体积——新功能倾向于落在独立 crate（[`codex-tools`](https://github.com/openai/codex/tree/main/codex-rs/tools)、[`ext/*`](https://github.com/openai/codex/tree/main/codex-rs/ext) 等）。下一节就是这些 crate 的地图。

---

## 重要目录

### 顶层布局

| 路径 | 用途 |
| ---- | ---- |
| [`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs) | Rust workspace（约 130 个 crate）——真正的产品，几乎所有实现都在这里 |
| [`codex-cli/`](https://github.com/openai/codex/tree/main/codex-cli) | 薄 npm 包（`@openai/codex`）；[`bin/codex.js`](https://github.com/openai/codex/blob/main/codex-cli/bin/codex.js) 只是 exec 平台原生 Rust 二进制 |
| [`sdk/`](https://github.com/openai/codex/tree/main/sdk) | 面向嵌入的 **TypeScript** 和 **Python** SDK |
| [`docs/`](https://github.com/openai/codex/tree/main/docs) | 用户/贡献者文档（多为指向 developers.openai.com 的占位） |
| [`scripts/`](https://github.com/openai/codex/tree/main/scripts)、[`tools/`](https://github.com/openai/codex/tree/main/tools)、[`bazel/`](https://github.com/openai/codex/tree/main/bazel)、[`patches/`](https://github.com/openai/codex/tree/main/patches)、[`justfile`](https://github.com/openai/codex/blob/main/justfile) | 构建、发布和开发工具 |
| [`third_party/`](https://github.com/openai/codex/tree/main/third_party) | vendored 依赖（如 V8） |

### [`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs) 内的关键 crate

**核心运行时**

- [`core`](https://github.com/openai/codex/tree/main/codex-rs/core) — 心脏：会话状态机、turn 循环、工具分发、历史/rollout、沙箱粘合
- [`protocol`](https://github.com/openai/codex/tree/main/codex-rs/protocol) — 各 crate 共享的词汇表：`Submission`、`Op`、`Event`、`ResponseItem`、`SandboxPolicy`、`AskForApproval`
- [`tools`](https://github.com/openai/codex/tree/main/codex-rs/tools) — core 与客户端共享的工具规格定义

**前端 / 入口**

- [`cli`](https://github.com/openai/codex/tree/main/codex-rs/cli) — `codex` 多合一二进制；解析子命令并分发
- [`tui`](https://github.com/openai/codex/tree/main/codex-rs/tui) — 交互式 Ratatui 终端 UI（无子命令时的默认）
- [`exec`](https://github.com/openai/codex/tree/main/codex-rs/exec) — 无头 `codex exec` 模式（CI/脚本）
- [`mcp-server`](https://github.com/openai/codex/tree/main/codex-rs/mcp-server) — 把 Codex 自身作为 MCP server 通过 stdio 暴露
- [`app-server`](https://github.com/openai/codex/tree/main/codex-rs/app-server) / [`app-server-protocol`](https://github.com/openai/codex/tree/main/codex-rs/app-server-protocol) — 支撑 IDE/桌面/SDK 的 JSON-RPC 服务 + v1/v2 API 类型

**模型 / API 层**

- [`codex-api`](https://github.com/openai/codex/tree/main/codex-rs/codex-api) — OpenAI Responses API 的 HTTP/WebSocket 客户端（SSE 解析）
- [`model-provider-info`](https://github.com/openai/codex/tree/main/codex-rs/model-provider-info) / [`model-provider`](https://github.com/openai/codex/tree/main/codex-rs/model-provider) — provider 元数据 + auth 解析
- [`models-manager`](https://github.com/openai/codex/tree/main/codex-rs/models-manager) — 模型目录
- [`ollama`](https://github.com/openai/codex/tree/main/codex-rs/ollama)、[`lmstudio`](https://github.com/openai/codex/tree/main/codex-rs/lmstudio) — 本地模型 provider 适配器

**认证**

- [`login`](https://github.com/openai/codex/tree/main/codex-rs/login) — `AuthManager`，OAuth（ChatGPT 登录）与 API key 认证
- [`chatgpt`](https://github.com/openai/codex/tree/main/codex-rs/chatgpt) — ChatGPT 后端能力（`codex apply`、connectors、cloud tasks）

**沙箱与安全**

- [`sandboxing`](https://github.com/openai/codex/tree/main/codex-rs/sandboxing) — 跨平台沙箱策略引擎（Seatbelt / seccomp+Landlock / Windows restricted token）
- [`linux-sandbox`](https://github.com/openai/codex/tree/main/codex-rs/linux-sandbox) — Linux 沙箱 helper 二进制（Landlock + seccomp + bwrap）
- [`execpolicy`](https://github.com/openai/codex/tree/main/codex-rs/execpolicy) — 用 Starlark 规则判定命令是否安全
- [`exec-server`](https://github.com/openai/codex/tree/main/codex-rs/exec-server) — 本地/远程进程执行服务

**工具积木**

- [`apply-patch`](https://github.com/openai/codex/tree/main/codex-rs/apply-patch) — `apply_patch` 文件编辑工具（也是独立的 arg0 入口）
- [`codex-mcp`](https://github.com/openai/codex/tree/main/codex-rs/codex-mcp) / [`rmcp-client`](https://github.com/openai/codex/tree/main/codex-rs/rmcp-client) — Codex 作为 MCP **客户端** 连接外部 server

**持久化与基础设施**

- [`config`](https://github.com/openai/codex/tree/main/codex-rs/config)、[`codex-home`](https://github.com/openai/codex/tree/main/codex-rs/codex-home) — `config.toml` 加载与 schema
- [`rollout`](https://github.com/openai/codex/tree/main/codex-rs/rollout) / [`rollout-trace`](https://github.com/openai/codex/tree/main/codex-rs/rollout-trace)、[`thread-store`](https://github.com/openai/codex/tree/main/codex-rs/thread-store)、[`state`](https://github.com/openai/codex/tree/main/codex-rs/state) — 会话持久化（SQLite + rollout 追踪）
- [`arg0`](https://github.com/openai/codex/tree/main/codex-rs/arg0) — 多合一二进制分发（`codex` 也能作为 `apply_patch` / `codex-linux-sandbox` 运行）
- [`otel`](https://github.com/openai/codex/tree/main/codex-rs/otel)、[`analytics`](https://github.com/openai/codex/tree/main/codex-rs/analytics) — 遥测

### 扩展 crate（[`codex-rs/ext/`](https://github.com/openai/codex/tree/main/codex-rs/ext)）

- [`connectors`](https://github.com/openai/codex/tree/main/codex-rs/ext/connectors) — 第三方服务集成
- [`extension-api`](https://github.com/openai/codex/tree/main/codex-rs/ext/extension-api) — 扩展注册和 hooks
- [`goal`](https://github.com/openai/codex/tree/main/codex-rs/ext/goal) — 目标跟踪扩展
- [`guardian`](https://github.com/openai/codex/tree/main/codex-rs/ext/guardian) — 审查/安全扩展
- [`image-generation`](https://github.com/openai/codex/tree/main/codex-rs/ext/image-generation) — 图像生成工具
- [`mcp`](https://github.com/openai/codex/tree/main/codex-rs/ext/mcp) — 托管 MCP server 集成（把 server 注入目录）
- [`memories`](https://github.com/openai/codex/tree/main/codex-rs/ext/memories) — 长期记忆
- [`skills`](https://github.com/openai/codex/tree/main/codex-rs/ext/skills) — Agent skills
- [`web-search`](https://github.com/openai/codex/tree/main/codex-rs/ext/web-search) — 网页搜索工具

---

## 请求如何流转

持久化的上下文按 **Thread → Turn → Item** 组织：thread 是一次对话，turn 是一轮用户↔代理交互，item 是一个上下文单元（一条消息、代理推理、一条 shell 命令、一次文件编辑）。

每个接入面都跑**同一套 core 循环**——一个 turn：

1. 输入作为 `Op::UserInput` 进入提交队列。
2. Session 组装模型上下文：系统指令、skills、git 信息、memories，以及可用工具。
3. `codex-api` 调用 **OpenAI Responses API**，SSE 流式接收 `ResponseEvent`。
4. 模型输出以 `Event` 流回——消息增量、推理、工具调用。
5. 遇到工具调用时，`ToolRouter` 分发到对应处理器（shell、`apply_patch`、文件搜索、MCP……）；命令在沙箱中运行，策略要求时**暂停等待审批**。
6. 工具结果回灌模型；第 3–6 步重复，直到 turn 结束。
7. Event 渲染到客户端；历史写入 thread store。

**三个接入面只在「输入如何到达、event 如何渲染」上有区别：**

- **TUI**（`codex`）——[`codex-tui`](https://github.com/openai/codex/tree/main/codex-rs/tui) 把输入/event 直接接到 core。
- **app-server**（`codex app-server`）——同一套循环，走 **JSON-RPC 2.0**（stdio/JSONL、unix socket 或 websocket），面向 IDE、桌面和 SDK。
- **无头**（`codex exec`）——提交 prompt，跑循环，打印结构化输出后退出。

app-server 消息交互：

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

### 执行路径中的沙箱

工具执行外包一层平台相关的隔离，由 `SandboxPolicy` 加审批 gate 约束：

| 平台 | 机制 |
|------|------|
| **macOS** | Seatbelt（`/usr/bin/sandbox-exec`） |
| **Linux** | bubblewrap（`bwrap`）+ Landlock/seccomp |
| **Windows** | Restricted-token sandbox（提升/非提升两种后端） |

在 OS 沙箱生效之前，[`execpolicy`](https://github.com/openai/codex/tree/main/codex-rs/execpolicy)（Starlark 规则引擎）会先判定命令是安全还是需要审批。`exec-server` 可本地运行，也可注册到远程 registry（WebSocket 上的 Noise 加密 relay），用于跨机器执行。

### 关键代码锚点

| 概念 | 位置 |
| ---- | ---- |
| `Codex` 句柄（队列对） | [`core/src/session/mod.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/session/mod.rs) |
| 提交循环（`Op` 分发） | [`core/src/session/handlers.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/session/handlers.rs) |
| Turn 执行（`run_turn`） | [`core/src/session/turn.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/session/turn.rs) |
| 模型客户端（SSE / WebSocket） | [`core/src/client.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/client.rs) |
| 工具路由 / 分发 | [`core/src/tools/router.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/tools/router.rs) |
| Shell 执行 + 沙箱选择 | [`core/src/exec.rs`](https://github.com/openai/codex/tree/main/codex-rs/core/src/exec.rs) |
| 多合一二进制分发（`arg0`） | [`arg0/src/lib.rs`](https://github.com/openai/codex/tree/main/codex-rs/arg0/src/lib.rs) |

---

## MCP：双向

Codex 在两个方向上都讲 **Model Context Protocol**。

- **作为 server**（[`mcp-server`](https://github.com/openai/codex/tree/main/codex-rs/mcp-server)）—— 一个 stdio JSON-RPC 循环，只暴露两个工具 `codex` 和 `codex-reply`。一次 `tools/call` 会拉起完整的 Codex 会话，提交 `Op::UserInput`，并把 core 的 `EventMsg` 作为 MCP 响应流式返回；审批请求变成 MCP **elicitation**。由此，任何别的代理都能驱动一个完整的 Codex 会话。
- **作为 client** —— 为了拿到更多工具，Codex 会连出去接外部 MCP server。[`rmcp-client`](https://github.com/openai/codex/tree/main/codex-rs/rmcp-client)（`RmcpClient`，stdio/HTTP + OAuth）是传输层；[`codex-mcp`](https://github.com/openai/codex/tree/main/codex-rs/codex-mcp)（`McpConnectionManager`）为每个 server 启一个 client，聚合它们的工具并处理命名冲突。turn 期间，这些工具注册为 `McpHandler`，调用沿 `handle_mcp_tool_call` → `McpConnectionManager::call_tool` → `RmcpClient` 路由。[`ext/mcp`](https://github.com/openai/codex/tree/main/codex-rs/ext/mcp) 通过 feature flag 注入托管 server（如 Codex Apps）。

---

## 技术栈

| 类别 | 技术 |
|------|------|
| **语言** | Rust（主）、TypeScript（SDK、npm 包装）、Python（SDK） |
| **异步运行时** | Tokio |
| **终端 UI** | ratatui + crossterm |
| **LLM API** | OpenAI Responses API（SSE 流式） |
| **协议** | JSON-RPC 2.0、JSONL、MCP（Model Context Protocol） |
| **持久化** | SQLite（threads）、rollout 追踪文件 |
| **沙箱** | macOS Seatbelt、Linux bubblewrap、Windows restricted-token sandbox |
| **远程执行** | WebSocket + Noise 加密 relay（exec-server 远程模式） |
| **构建** | Cargo workspace + Bazel；`just` 用于开发命令 |
| **包管理** | pnpm（Node）、Cargo（Rust） |
| **测试** | `insta` 快照测试（TUI）、`core/suite` 集成测试 |
| **可观测性** | OpenTelemetry（`codex-otel`）、结构化 tracing |
| **Schema** | 配置的 JSON Schema；app-server API 的 TypeScript 生成 |

---

## 心智模型

四个协作系统，都围绕 core 的 session/turn 循环组合：

1. **对话引擎** — threads、turns、上下文组装、模型流式输出
2. **工具运行时** — shell、补丁、搜索、MCP、扩展
3. **安全层** — 沙箱、执行策略、审批
4. **集成面** — TUI、app-server JSON-RPC、SDK、`exec` 模式

`codex` 二进制是唯一入口；其余都遵循同一个节奏：**流式模型响应 → 执行工具 → 持久化状态 → 发出 event。**

---

## 延伸阅读

### 本仓库（codex-labs）

- [layeredDesign_cn.md](layeredDesign_cn.md) / [layeredDesign.md](layeredDesign.md) — 分层架构详解
- [resources/links_zh.md](../resources/links_zh.md) / [links.md](../resources/links.md) — 精选外部链接

### GitHub 上的 [openai/codex](https://github.com/openai/codex)

- [README.md](https://github.com/openai/codex/blob/main/README.md) — 项目概览与快速开始
- [AGENTS.md](https://github.com/openai/codex/blob/main/AGENTS.md) — 贡献者约定和 crate 指引
- [docs/contributing.md](https://github.com/openai/codex/blob/main/docs/contributing.md) — 贡献指南
- [docs/install.md](https://github.com/openai/codex/blob/main/docs/install.md) — 安装与构建
- [codex-rs/app-server/README.md](https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md) — app-server 协议和 API 参考
- [codex-rs/core/README.md](https://github.com/openai/codex/blob/main/codex-rs/core/README.md) — 各平台沙箱要求
- [codex-rs/codex-api/README.md](https://github.com/openai/codex/blob/main/codex-rs/codex-api/README.md) — Responses API 客户端接口
- [codex-rs/exec-server/README.md](https://github.com/openai/codex/blob/main/codex-rs/exec-server/README.md) — 进程执行服务
- [codex-rs/protocol/README.md](https://github.com/openai/codex/blob/main/codex-rs/protocol/README.md) — 共享协议类型
- [codex-rs/tools/README.md](https://github.com/openai/codex/blob/main/codex-rs/tools/README.md) — 工具规格与契约

### 官方文档

- [Codex 官方文档](https://developers.openai.com/codex)

---

_基于对 [openai/codex](https://github.com/openai/codex) 仓库的探索整理。_
