# Codex 仓库架构总览

[English](architecture.md) | **中文**

[openai/codex](https://github.com/openai/codex) 仓库的高层概览——项目做什么、如何组织、请求如何流转、用了哪些技术。

> 逐层详解见 [layeredDesign_cn.md](layeredDesign_cn.md)（中文）或 [layeredDesign.md](layeredDesign.md)（English）。

---

## 项目做什么

**Codex CLI** 是 OpenAI 的本地编程代理。它运行在你的机器上，可以：

- 接受自然语言任务（「修这个测试」「重构这个模块」等）
- 用 LLM 推理 + 内置工具来规划和执行工作
- 读写文件、跑 shell 命令、搜索网页、调用 MCP 工具
- 在可配置的沙箱和审批策略下运行
- 把对话历史持久化为 **thread（会话）** 和 **turn（轮次）**

发布的二进制叫 `codex`。npm 包 `@openai/codex` 只是调用该原生二进制的薄包装。

相关产品（不都在本仓库里）：

- **Codex in IDE** — 通过 app-server 协议接入 VS Code、Cursor、Windsurf
- **Codex App** — 桌面体验（`codex app`）
- **Codex Web** — 云端代理，见 chatgpt.com/codex

---

## 整体架构

仓库是一个 **Rust 优先的 monorepo**（[`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs)），约 100 个 crate，外加 Node/TypeScript 打包和 SDK。

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
    │               │ │               │ │               │ │               │
    │               │ │               │ │               │ │               │
    │               │ │               │ │               │ │               │
    └───────────────┘ └───────────────┘ └───────────────┘ └───────────────┘
```

> **说明：** 图中 crate 名与英文标签保留英文，以便在等宽字体下对齐。中文标题仅用于分区标识。

### 分层设计

| 层 | Crate | 职责 |
| ---- | ----- | ---- |
| **入口** | [`codex-cli`](https://github.com/openai/codex/tree/main/codex-rs/cli) | `codex` 二进制路由到 TUI、`app-server`、`exec`、MCP、插件等 |
| **协议** | [`codex-protocol`](https://github.com/openai/codex/tree/main/codex-rs/protocol) | 共享的 op、event、配置、thread/turn item 类型 |
| **核心** | [`codex-core`](https://github.com/openai/codex/tree/main/codex-rs/core) | 代理循环、会话管理、上下文构建、工具编排 |
| **API 客户端** | [`codex-api`](https://github.com/openai/codex/tree/main/codex-rs/codex-api)、[`codex-client`](https://github.com/openai/codex/tree/main/codex-rs/codex-client) | 与 OpenAI Responses API 通信（SSE 流式） |
| **执行** | [`exec-server`](https://github.com/openai/codex/tree/main/codex-rs/exec-server) | 拉起和控制子进程；可本地或远程 |
| **沙箱** | [`sandboxing`](https://github.com/openai/codex/tree/main/codex-rs/sandboxing)、[`linux-sandbox`](https://github.com/openai/codex/tree/main/codex-rs/linux-sandbox)、[`windows-sandbox-rs`](https://github.com/openai/codex/tree/main/codex-rs/windows-sandbox-rs) | 平台相关的隔离机制 |
| **扩展** | [`ext/*`](https://github.com/openai/codex/tree/main/codex-rs/ext) | Skills、MCP、connectors、memories、网页搜索、goal 等 |
| **IDE 集成** | [`app-server`](https://github.com/openai/codex/tree/main/codex-rs/app-server)、[`app-server-protocol`](https://github.com/openai/codex/tree/main/codex-rs/app-server-protocol) | 面向 VS Code 扩展等富客户端的 JSON-RPC API |

[`codex-core`](https://github.com/openai/codex/tree/main/codex-rs/core) 是中心编排器，但仓库有意控制其体积；新功能倾向于落在独立 crate（[`codex-tools`](https://github.com/openai/codex/tree/main/codex-rs/tools)、[`ext/*`](https://github.com/openai/codex/tree/main/codex-rs/ext) 等）。

### 核心抽象：队列对

`codex-core` 把 `Codex` 暴露为高层句柄，以 **队列对** 运作：调用方发送 `Op`，接收 `Event` 流。

```rust
pub struct Codex {
    pub(crate) tx_sub: Sender<Submission>,
    pub(crate) rx_event: Receiver<Event>,
    pub(crate) agent_status: watch::Receiver<AgentStatus>,
    pub(crate) session: Arc<Session>,
    // ...
}
```

主要入站操作（`Op`）：`UserInput`、`ExecApproval`、`PatchApproval`、`Interrupt`、`ThreadSettings` 等。

主要出站消息（`Event` / `EventMsg`）：代理消息增量、工具进度、审批请求、turn 完成、错误等。

---

## 重要目录

| 路径 | 用途 |
| ---- | ---- |
| [`codex-rs/`](https://github.com/openai/codex/tree/main/codex-rs) | 主 Rust workspace，几乎所有实现都在这里 |
| [`codex-rs/core/`](https://github.com/openai/codex/tree/main/codex-rs/core) | 代理业务逻辑：会话、turn、工具、上下文、审批 |
| [`codex-rs/cli/`](https://github.com/openai/codex/tree/main/codex-rs/cli) | `codex` 二进制入口和子命令路由 |
| [`codex-rs/tui/`](https://github.com/openai/codex/tree/main/codex-rs/tui) | 终端 UI（ratatui） |
| [`codex-rs/app-server/`](https://github.com/openai/codex/tree/main/codex-rs/app-server) | 面向 IDE/桌面的 JSON-RPC 服务 |
| [`codex-rs/app-server-protocol/`](https://github.com/openai/codex/tree/main/codex-rs/app-server-protocol) | v1/v2 API 类型（Rust + 生成的 TypeScript schema） |
| [`codex-rs/protocol/`](https://github.com/openai/codex/tree/main/codex-rs/protocol) | core、TUI、app-server 共享的内部协议类型 |
| [`codex-rs/codex-api/`](https://github.com/openai/codex/tree/main/codex-rs/codex-api) | OpenAI Responses API 客户端和 SSE 解析 |
| [`codex-rs/exec-server/`](https://github.com/openai/codex/tree/main/codex-rs/exec-server) | 本地/远程进程执行服务 |
| [`codex-rs/sandboxing/`](https://github.com/openai/codex/tree/main/codex-rs/sandboxing) | 跨平台沙箱策略与强制执行 |
| [`codex-rs/tools/`](https://github.com/openai/codex/tree/main/codex-rs/tools) | 工具规格、适配器和共享执行契约 |
| [`codex-rs/ext/`](https://github.com/openai/codex/tree/main/codex-rs/ext) | 可选扩展：skills、MCP、connectors、memories、web-search 等 |
| [`codex-rs/config/`](https://github.com/openai/codex/tree/main/codex-rs/config) | `config.toml` 加载和 schema |
| [`codex-rs/thread-store/`](https://github.com/openai/codex/tree/main/codex-rs/thread-store) | thread/turn 持久化（SQLite） |
| [`codex-rs/rollout/`](https://github.com/openai/codex/tree/main/codex-rs/rollout) | 会话 rollout 追踪，用于调试/回放 |
| [`codex-cli/`](https://github.com/openai/codex/tree/main/codex-cli) | npm 包（`@openai/codex`）—— 分发包装 |
| [`sdk/`](https://github.com/openai/codex/tree/main/sdk) | 嵌入 Codex 用的 TypeScript 和 Python SDK |
| [`docs/`](https://github.com/openai/codex/tree/main/docs) | 贡献者文档（安装、配置、沙箱、贡献指南） |
| [`scripts/`](https://github.com/openai/codex/tree/main/scripts)、[`bazel/`](https://github.com/openai/codex/tree/main/bazel)、[`justfile`](https://github.com/openai/codex/blob/main/justfile) | 构建、测试和开发自动化 |
| [`third_party/`](https://github.com/openai/codex/tree/main/third_party) | vendored 依赖（如 V8） |

### 扩展 crate（[`codex-rs/ext/`](https://github.com/openai/codex/tree/main/codex-rs/ext)）

- [`connectors`](https://github.com/openai/codex/tree/main/codex-rs/ext/connectors) — 第三方服务集成
- [`extension-api`](https://github.com/openai/codex/tree/main/codex-rs/ext/extension-api) — 扩展注册和 hooks
- [`goal`](https://github.com/openai/codex/tree/main/codex-rs/ext/goal) — 目标跟踪扩展
- [`guardian`](https://github.com/openai/codex/tree/main/codex-rs/ext/guardian) — 审查/安全扩展
- [`image-generation`](https://github.com/openai/codex/tree/main/codex-rs/ext/image-generation) — 图像生成工具
- [`mcp`](https://github.com/openai/codex/tree/main/codex-rs/ext/mcp) — MCP 服务集成
- [`memories`](https://github.com/openai/codex/tree/main/codex-rs/ext/memories) — 长期记忆
- [`skills`](https://github.com/openai/codex/tree/main/codex-rs/ext/skills) — Agent skills
- [`web-search`](https://github.com/openai/codex/tree/main/codex-rs/ext/web-search) — 网页搜索工具

---

## 请求如何流转

### 核心概念（app-server 模型）

API 暴露三个顶层概念：

- **Thread** — 用户与代理之间的一次对话；包含多个 turn
- **Turn** — 一轮交互，通常是用户输入 → 代理响应；包含多个 item
- **Item** — 持久化的上下文单元：用户消息、代理推理、shell 命令、文件编辑等

### 交互式 TUI（`codex`）

1. 用户在终端 UI（`codex-tui`）输入 prompt。
2. TUI 发送 **turn start** 和用户输入（经 app-server 或直接连 core）。
3. Core 在异步提交队列收到 `Op::UserInput`。
4. **Session** 构建模型上下文：系统指令、skills、插件、git 信息、memories、MCP 工具等。
5. Core 通过 `codex-api` 调用 **OpenAI Responses API**，SSE 流式接收 `ResponseEvent`。
6. 模型流式输出时，core 发出 **event**（消息增量、推理、工具调用等）。
7. 模型请求 **tool call**（shell、`apply_patch`、文件搜索、MCP 等）时：
   - `ToolRouter` 路由到对应处理器
   - **审批**可能暂停等待用户确认（执行命令、打补丁、网络访问）
   - 命令经 **exec-server** 在 **沙箱策略** 下运行
8. 工具结果回灌模型；循环直到 turn 结束。
9. Event 流式推到 UI；历史写入 **thread store**。

### IDE / SDK（`codex app-server`）

同样的 core 循环，但通过 **JSON-RPC 2.0** 走 stdio（JSONL）、unix socket 或 websocket：

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

生命周期：

1. **Initialize** — 客户端发 `initialize` 和 `clientInfo`；服务端返回能力
2. **Start/resume thread** — `thread/start`、`thread/resume` 或 `thread/fork`
3. **Begin turn** — `turn/start`，带 `threadId` 和用户输入
4. **Stream events** — 通知：`item/started`、`item/completed`、`item/agentMessage/delta`、工具进度
5. **Finish turn** — `turn/completed`，含最终状态和 token 用量

### 无头模式（`codex exec`）

面向 CI/脚本的非交互模式：提交 prompt，跑代理循环，打印结构化输出后退出。

### 执行路径中的沙箱

工具执行外包一层平台相关的强制隔离：

| 平台 | 机制 |
|------|------|
| **macOS** | Seatbelt（`/usr/bin/sandbox-exec`） |
| **Linux** | bubblewrap（`bwrap`），旧策略下可回退 Landlock |
| **Windows** | Restricted-token sandbox（提升/非提升两种后端） |

执行命令和文件系统访问由 `SandboxPolicy` / permissions 配置约束。用户审批可以阻止或放行单次操作。

### 远程执行

`exec-server` 可本地运行（WebSocket），也可注册到远程环境 registry。远程模式通过 WebSocket 上的 **Noise 加密 relay 帧** 跨机器控制进程——适合 app-server 和 exec-server 跑在不同 OS 上的场景。

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

可以把 Codex 想成四个协作系统：

1. **对话引擎** — threads、turns、上下文组装、模型流式输出
2. **工具运行时** — shell、补丁、搜索、MCP、扩展
3. **安全层** — 沙箱、执行策略、审批
4. **集成面** — TUI、app-server JSON-RPC、SDK、`exec` 模式

`codex` 二进制是唯一入口。其余都围绕 core 的 session/turn 循环组合：流式模型响应 → 执行工具 → 持久化状态 → 发出 event。

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