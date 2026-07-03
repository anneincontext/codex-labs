# 案例：Realtime AI Presenter demo

[English](case.md) | **中文**

**类型：** 自己的 demo（Anne 与 Codex 的会话）  
**来源：** Anne 自己做 demo

## Skills / Workflow / Prompts

- **Prompt：** [prompts/implement-demo_zh.md](../../prompts/implement-demo_zh.md)
- **持久规则：** [AGENTS.md](https://github.com/anneincontext/realtime-ai-presenter/blob/main/AGENTS.md)
- **仓库：** [anneincontext/realtime-ai-presenter](https://github.com/anneincontext/realtime-ai-presenter)

## 背景

在 OpenAI realtime-console 上做可打断的语音讲解 demo——幻灯片、function calling 导航、偏产品感的 UI。

## 我向 Codex 提了什么

一份结构化 prompt：目标、基础仓库、演示交互、技术偏好、成功标准、约束、验收。见 [implement-demo_zh.md](../../prompts/implement-demo_zh.md)。

## 时间线

| 阶段     | 发生了什么                                                                                       | 约耗时       |
| -------- | ------------------------------------------------------------------------------------------------ | ------------ |
| **启动** | 一份结构化 prompt（[implement-demo_zh.md](../../prompts/implement-demo_zh.md)）→ 可运行 demo     | **约 20 分钟** |
| **打磨** | 持续对话 — 可用性、边界情况、README 捷径、讲解流程                                               | 持续进行     |

第一遍能很快「能演示」，是因为 prompt 里已经写清基础仓库、成功标准和验收方式；后续回合多是**小步追问**，不用重复讲 entire 产品。

## Codex 做了什么

在 `openai-realtime-console`（WebRTC、音频、data channel）上扩展成 slide presenter；并留下 `AGENTS.md` 供后续会话使用。

## 哪些有效

- **约 20 分钟到第一版 demo** — 结构化 prompt 预先拍板（不重写、完成标准清晰）
- **Prompt 分段直接对应决策** — Base repo 挡住重写；Success criteria 就是完成定义
- **先启动再迭代** — 骨架用强 prompt，打磨用短 follow-up，比万事塞一个巨型 prompt 省事
- **启动后用短 AGENTS.md** — 打磨阶段每次进上下文成本低
- **「扩展而非重写」** 同时出现在 prompt Constraints 和 AGENTS.md — 信息一致

## 哪些地方需要我引导

- 语音/幻灯片的手动验收（Verification）— 打磨阶段尤其需要
- 是否批准新的生产依赖（AGENTS.md working agreement）
- 判断「像产品 demo」的程度何时够好

## 收获

```text
阶段 A（~20 分钟）：prompt.md     →  能演示的骨架（做什么 / 怎么算完成 / 怎么验）
阶段 B（持续）：    对话迭代      →  可用性 + 体验
阶段 C（每次改动）：AGENTS.md     →  习惯 / 品味 / 边界
```

**模式：** 一次强 bootstrap prompt，再多轮小步掌舵 — 不要用一个大 prompt 扛整个项目。

完整模型见 [notes/agents-md_cn.md](../../notes/agents-md_cn.md)。