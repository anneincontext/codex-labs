# Codex 分层架构详解

[English](layeredDesign.md) | **中文**

基于 [architecture_cn.md](architecture_cn.md) 中的 **分层设计**，逐层讲解 Codex 仓库的架构。

---

## 总览：各层如何串联

可以把整体想成：**上面是入口，中间是协议和大脑，下面是能力和安全边界**。

```text
你（用户 / IDE / SDK）
        │
        ▼
  ① Entry points / ⑧ IDE integration   ← 怎么进来
        │
        ▼
  ② Protocol                            ← 大家说什么「语言」
        │
        ▼
  ③ Core                                ← 代理怎么思考、怎么跑一圈
        │
   ┌────┴────┬──────────┬──────────┐
   ▼         ▼          ▼          ▼
⑦ Extensions  ④ API    ⑤ Execution  ⑥ Sandboxing
```

---

## ① Entry points — `codex-cli`

**角色：** 整个系统的**唯一可执行入口**（`codex` 命令）。

你在终端敲 `codex`，实际跑的是 `codex-rs/cli` 编译出来的二进制。它**不做**代理推理本身，只做**路由**：

| 子命令 / 模式 | 干什么 |
|---------------|--------|
| 默认（无参数） | 进 **TUI** 交互界面 |
| `codex exec` | **无头模式**，适合脚本 / CI |
| `codex app-server` | 起 **JSON-RPC 服务**，给 IDE / SDK 用 |
| `codex mcp` / `plugin` / `login` 等 | 配置、认证、插件管理 |

npm 上的 `@openai/codex` 只是**包装器**，最终还是调这个原生二进制。

**这一层的价值：** 把「怎么用 Codex」统一成一个命令，下面各模式共享同一套 core。

---

## ② Protocol — `codex-protocol`

**角色：** 各组件之间的**共同语言**——只定义类型，几乎不写业务逻辑。

这里是 core、TUI、app-server 共用的**协议类型层**，依赖要尽量少。

核心概念：

| 类型 | 方向 | 含义 |
|------|------|------|
| `Op` | 调用方 → Core | 你要 Core **做什么**（用户输入、审批、中断等） |
| `Event` / `EventMsg` | Core → 调用方 | Core **发生了什么**（流式输出、工具进度、要审批了） |
| `ThreadId`, `TurnItem`, `Settings` 等 | 双向 | 会话、配置、持久化结构的公共定义 |

例如 `Op::UserInput` 表示「开始一轮对话」；`EventMsg` 里会有消息增量、工具调用、turn 结束等。

**这一层的价值：** TUI、app-server、exec 不用各自发明一套消息格式，都通过 `Op` / `Event` 和 core 对话。改 UI 不用改 core 的内部实现，只要协议兼容。

---

## ③ Core — `codex-core`

**角色：** **代理的大脑和调度中心**——会话、上下文、模型调用、工具编排都在这里。

最核心的抽象是 **queue pair（队列对）**：

```text
调用方                    codex-core
   │                          │
   │── submit(Op) ───────────►│  提交队列
   │                          │  session loop 处理
   │◄── Event stream ─────────│  事件队列
```

一次 **Turn（轮次）** 在 core 里大致是：

1. 收到 `Op::UserInput`
2. **拼上下文**：系统指令、skills、插件、git 信息、memories、MCP 工具列表……
3. 通过 `codex-api` 调 **OpenAI Responses API**，SSE 流式收模型输出
4. 解析流里的 **tool call**，交给 `ToolRouter` 执行
5. 需要时发 **审批请求**（跑 shell、改文件、访问网络），等用户 `Op::ExecApproval` 等
6. 工具结果塞回模型，循环直到 turn 结束
7. 通过 `Event` 往外推；历史写入 **thread-store**

**这一层的价值：** 所有「代理怎么工作」的规则集中在这里。上层只关心「发 Op、收 Event」，不关心模型怎么调、工具怎么路由。

> 仓库有意**控制 core 体积**：新功能尽量放到 `codex-tools`、`ext/*` 等独立 crate，core 负责编排。

---

## ④ API client — `codex-api` + `codex-client`

**角色：** 和 **OpenAI Responses API** 的**网络层**——发请求、收 SSE、重试、鉴权头。

分工：

| Crate | 职责 |
|-------|------|
| `codex-client` | 通用 HTTP 传输 |
| `codex-api` | Responses / Compact / Memory 等 API 的请求体、SSE 解析、`ResponseEvent` 流 |

core 拼好 `instructions`、`input`、`tools` 后，交给这一层；收回来的是结构化的 `ResponseEvent`（文本增量、function call、完成信号等）。

**这一层的价值：** 把「和 OpenAI 怎么说话」从业务逻辑里剥离。换 endpoint、加重试、改 SSE 解析，主要动这一层，不动 session / turn 逻辑。

---

## ⑤ Execution — `exec-server`

**角色：** **真正在机器上跑进程**的地方——shell、PTY、子进程生命周期。

- 本地：`codex exec-server`，WebSocket 上 JSON-RPC
- 远程：注册到环境 registry，通过 **Noise 加密 relay** 跨机器控制进程（app-server 和 exec-server 可以在不同 OS 上）

core 决定「要跑 `cargo test`」；**exec-server** 负责 spawn、传 stdin/stdout、管理后台终端等。

**这一层的价值：** 把「危险的操作」（起子进程）从 core 里拆出去，便于本地/远程部署，也便于单独加固。

---

## ⑥ Sandboxing — `sandboxing` 等

**角色：** **安全边界**——限制文件系统和网络能碰哪里。

| 平台 | 机制 |
|------|------|
| macOS | Seatbelt (`sandbox-exec`) |
| Linux | bubblewrap (`bwrap`)，部分场景 Landlock |
| Windows | restricted-token sandbox |

策略来自 `SandboxPolicy` / permissions 配置（只读、工作区可写、网络规则等）。exec-server 起命令前，会按策略包一层沙箱。

**这一层的价值：** 模型再聪明，真正动你磁盘和网络前，还有一层**平台级隔离** + 上面的**用户审批**。

---

## ⑦ Extensions — `ext/*`

**角色：** **可插拔能力**，不把所有东西塞进 core。

| 扩展 | 能力 |
|------|------|
| `ext/skills` | Agent skills |
| `ext/mcp` | MCP 工具服务 |
| `ext/connectors` | 第三方服务连接 |
| `ext/memories` | 长期记忆 |
| `ext/web-search` | 网页搜索 |
| `ext/guardian` | 审查 / 安全 |
| `ext/goal` | 目标跟踪 |

core 在拼上下文、注册工具时会**拉这些扩展进来**，但具体实现留在各自 crate。

**这一层的价值：** 新能力可以独立开发、测试、开关，符合仓库「少胖 core」的方向。

---

## ⑧ IDE integration — `app-server` + `app-server-protocol`

**角色：** 给 **富客户端**用的**对外 API 层**（VS Code 插件、桌面 App、TypeScript/Python SDK）。

和 TUI 不同，这一层不画终端 UI，而是：

- 对外：**JSON-RPC 2.0**（stdio JSONL、unix socket 等）
- 对内：还是 `Op` / `Event` 那套，通过 `codex-core` 跑代理

对外概念（比内部 `Op` 更产品化）：

```text
Thread（会话）→ Turn（一轮）→ Item（消息 / 命令 / 编辑 / 推理片段）
```

典型流程：

```text
initialize → thread/start → turn/start → 收 item/* 通知 → turn/completed
```

`app-server-protocol` 定义 v2 API 类型，并可生成 TypeScript schema，保证 IDE 和 CLI 版本对齐。

**这一层的价值：** 把 core 包装成**稳定、可版本化的 RPC 接口**，让编辑器、SDK 不用嵌入 Rust TUI。

---

## 串起来：一层请求怎么走

以 VS Code 里发一句「修这个测试」为例：

```text
1. VS Code 扩展
      │  JSON-RPC: turn/start
      ▼
2. app-server（⑧）
      │  转成 Op::UserInput
      ▼
3. codex-protocol（②）  类型对齐
      ▼
4. codex-core（③）
      │  拼上下文，拉 ext/skills、mcp（⑦）
      │  调 codex-api（④）→ OpenAI
      │  模型要跑测试 → ToolRouter
      ▼
5. exec-server（⑤）  spawn shell
      │  外包一层 sandboxing（⑥）
      ▼
6. 结果回 core → 再喂模型 → Event 流
      ▼
7. app-server → item/completed 等通知 → VS Code 显示
```

---

## 小结：每层一句话

| 层 | 一句话 |
|----|--------|
| Entry points | `codex` 命令，选 TUI / exec / app-server |
| Protocol | 大家共用的 Op / Event / 配置类型 |
| Core | 代理主循环：上下文、模型、工具、持久化 |
| API client | 和 OpenAI Responses API 通信 |
| Execution | 在本地或远程真正跑子进程 |
| Sandboxing | 限制文件和网络访问 |
| Extensions | skills、MCP、记忆等可插拔能力 |
| IDE integration | JSON-RPC 门面，给编辑器和 SDK 用 |

---

## 延伸阅读

- [architecture_cn.md](architecture_cn.md) — 仓库架构总览
- [openai/codex](https://github.com/openai/codex) — 源码仓库
- [Codex 官方文档](https://developers.openai.com/codex)