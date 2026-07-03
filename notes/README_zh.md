# Notes

[English](README.md) | **中文**

理论、心智模型和研究笔记——**为什么** Codex 是这样工作的。

不是会话复盘（`cases/`），也不是可直接跑的模板（`prompts/`）。

## 目录组织

```text
notes/
├── codex/      # 「openai/codex 是怎么做的？」— 研究对象笔记
├── method/     # 「我该怎么和 agent 协作？」— 可迁移方法论
└── research/   # 「别人在怎么用 AI？」    — 田野调查
```

每篇笔记开头都有档案头：**回答什么**（解决什么问题）、**读者前置**（先读什么），`codex/` 类笔记额外带**校验基准**（结论核对时的上游 commit；代码笔记会过期，方法论笔记不会）。

## 从这里开始——按目的选

| 你想… | 读这个 |
| ----- | ------ |
| 快速建立 openai/codex 全貌 | [codex/architecture_zh.md](codex/architecture_zh.md) · [EN](codex/architecture.md) *(hub——其他 codex/ 笔记都从它分出)* |
| 理解 crate 分层 | [codex/layered-design_zh.md](codex/layered-design_zh.md) · [EN](codex/layered-design.md) |
| 看终端 UI 怎么设计的 | [codex/tui-interface-design_zh.md](codex/tui-interface-design_zh.md) · [EN](codex/tui-interface-design.md) |
| 查 TUI slash 命令 / 快捷键 | [codex/tui-commands_zh.md](codex/tui-commands_zh.md) · [EN](codex/tui-commands.md) |
| 学「用 agent 读懂一个仓库」（本 lab 的工作流） | [method/learn-codebase-flow_zh.md](method/learn-codebase-flow_zh.md) · [EN](method/learn-codebase-flow.md) |
| 写好一份 AGENTS.md | [method/agents-md_zh.md](method/agents-md_zh.md) · [EN](method/agents-md.md) |
| 了解从业者实际怎么用 AI 工具 | [research/real-workflows_zh.md](research/real-workflows_zh.md) · [EN](research/real-workflows.md) |

## 约定

- **英文版为准，`_zh.md` 镜像跟随。** 同一次改动里两边一起更新。
- **Hub-and-spoke：** 深潜笔记只链回自己的 hub（`codex/architecture_zh.md`），不要每篇互链。
- **`codex/` 笔记带「校验基准」commit** ——引用细节前，先在上游跑 `git diff <commit>..HEAD -- codex-rs/` 看有没有漂移。
