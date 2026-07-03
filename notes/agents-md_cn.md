# AGENTS.md 约束了什么？

[English](agents-md.md) | **中文**

`AGENTS.md` 是**项目本地的约定文件**，会进入 agent 的上下文。它不执行代码——而是在工作前/工作中**塑造决策**。

## 示例：[realtime-ai-presenter/AGENTS.md](https://github.com/anneincontext/realtime-ai-presenter/blob/main/AGENTS.md)

文件很短（约 25 行），但约束了四个层次：

### 1. Working agreements（硬规则）

| 规则 | 约束什么 |
| ---- | -------- |
| 改 JavaScript 后跑 `npm test` | **验证** — 不能只改完就停 |
| 装依赖优先 `pnpm` | **工具链** — 包管理器选择 |
| 加生产依赖前先问 | **审批关卡** — 依赖膨胀需人同意 |

最接近**可检查的策略**：具体、能验证、往往很机械。

### 2. Principles — 优先

| 原则 | 约束什么 |
| ---- | -------- |
| 可读代码 | 风格与命名 — 清晰优于炫技 |
| 低 agent 决策成本 | 架构 — 少让 agent 猜分支 |
| 小且独立的模块 | 结构 — 文件/模块粒度 |
| 未来可扩展 | API 形态 — 留接缝、避免死胡同 |
| 对开发者友好 | DX — 文档、错误信息、可预测布局 |

在多种实现都「能跑」时，用来**打破平局**。

### 3. Principles — 避免

| 反模式 | 约束什么 |
| ------ | -------- |
| 大型抽象 | 不随意引入框架/厚封装 |
| 复杂状态管理 | 倾向局部、简单状态 |
| 过早优化 | 先正确/清晰再谈性能 |
| 沉重依赖 | `package.json` 与供应链重量 |

**否定式约束** — agent 默认不该伸手去拿的东西。

### 4. 改动策略

| 规则 | 约束什么 |
| ---- | -------- |
| 扩展现有代码，而非重写 | **改动范围** — 修补生长，别整片替换 |
| 简单、无聊的方案 | **方案类别** — 拒绝花哨设计 |

## 心智模型

```text
AGENTS.md
├── Working agreements   →  可检查的习惯（测试、pnpm、先问）
├── Priorities           →  设计时的取舍标准
├── Avoid list           →  默认「不要」清单
└── Change strategy      →  怎么动代码库
```

对比 [openai/codex 的 AGENTS.md](https://github.com/openai/codex/blob/main/AGENTS.md)：同一思路，体量大得多——crate 规则、沙箱环境变量、测试命令、Clippy、app-server API 形态等。

## 与 codex-labs 的对应

| AGENTS.md 层次 | 在本 lab 的位置 |
| -------------- | --------------- |
| 全仓库规则 | [`AGENTS.md`](../AGENTS.md) |
| 单项 skill 步骤 | `skills/<name>/SKILL.md` |
| 人工关卡 | `skills/<name>/checklist.md` |
| 多 skill 流程 | `workflows/<name>/WORKFLOW.md` |
| 触发模板 | `prompts/<name>.md` |

你的 presenter demo 说明：**稳定、项目相关的约束，用短 AGENTS.md 往往比长 system prompt 更有效**。

## prompt.md 与 AGENTS.md 分工（Realtime 案例）

Anne 用 [`Realtime api 的 prompt.md`](../../prompts/implement-demo_zh.md) **启动** demo，用 [AGENTS.md](https://github.com/anneincontext/realtime-ai-presenter/blob/main/AGENTS.md) **维护** demo：

| 文件 | 何时用 | 约束什么 |
| ---- | ------ | -------- |
| `prompt.md` | 第 1 次（或大功能） | 产品目标、基础仓库、成功标准、验收命令、范围上限 |
| `AGENTS.md` | 每次后续编辑 | 测试习惯、工具链、加依赖审批、代码品味、反模式 |

案例复盘：[cases/001-realtime-ai-presenter/case_zh.md](../cases/001-realtime-ai-presenter/case_zh.md)。