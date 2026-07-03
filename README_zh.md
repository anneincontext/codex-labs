# Codex Labs

[English](README.md) | **中文**

我和 Codex 共用的学习仓库——技能、流程、案例与笔记。

## 项目结构

```text
codex-labs/
├── skills/          # 仅 _template/ — 从真实 case 沉淀后再加
├── workflows/       # 仅 _template/ — skill 能串起来后再加
├── prompts/
├── cases/
│   ├── _template/
│   ├── 001-realtime-ai-presenter/
│   └── …
├── experiments/
├── notes/
│   ├── codex/       # 研究 openai/codex
│   ├── method/      # 可迁移方法论
│   └── research/    # 田野调查
└── resources/
```

## 如何串起来

```text
prompts  ──触发──►  skills  ──组合──►  workflows
cases / experiments ◄────────── 记录真实运行
notes / resources   ─────────── 原理与参考资料
```

## Agent 约束

| 文档               | English                                  | 中文                                   |
| ------------------ | ---------------------------------------- | -------------------------------------- |
| 本 lab 工作约定    | [AGENTS.md](AGENTS.md)                   | [AGENTS_zh.md](AGENTS_zh.md)           |
| AGENTS.md 约束什么 | [notes/method/agents-md.md](notes/method/agents-md.md) | [notes/method/agents-md_zh.md](notes/method/agents-md_zh.md) |

## 从这里开始

### Notes

| 主题                               | English                                                        | 中文                                                           |
| ---------------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------- |
| 学习代码库（prompt → 结果）        | [notes/method/learn-codebase-flow.md](notes/method/learn-codebase-flow.md)   | [notes/method/learn-codebase-flow_zh.md](notes/method/learn-codebase-flow_zh.md) |
| Codex 架构                         | [notes/codex/architecture.md](notes/codex/architecture.md)                | [notes/codex/architecture_zh.md](notes/codex/architecture_zh.md)            |
| 分层架构                           | [notes/codex/layered-design.md](notes/codex/layered-design.md)             | [notes/codex/layered-design_zh.md](notes/codex/layered-design_zh.md)         |
| AGENTS.md 约束什么                 | [notes/method/agents-md.md](notes/method/agents-md.md)                     | [notes/method/agents-md_zh.md](notes/method/agents-md_zh.md)                 |
| 田野调研（三次语音访谈）           | [notes/research/real-workflows.md](notes/research/real-workflows.md)           | [notes/research/real-workflows_zh.md](notes/research/real-workflows_zh.md)       |

### 案例

| 案例                         | 简介                                                                    |
| ---------------------------- | ----------------------------------------------------------------------- |
| [005-learn-codex-repo](cases/005-learn-codex-repo/case_zh.md) | 当前这个 repo 能够存在的原因 · [English](cases/005-learn-codex-repo/case.md) |
| [001-realtime-ai-presenter](cases/001-realtime-ai-presenter/case_zh.md) | 用 Codex 做 OpenAI Realtime API 的 demo · [English](cases/001-realtime-ai-presenter/case.md) |
| [002-biotech-swe](cases/002-biotech-swe/case_zh.md) | 生物科技公司工程师的真实感受 · [English](cases/002-biotech-swe/case.md) |
| [003-fintech-doc-pipeline](cases/003-fintech-doc-pipeline/case_zh.md) | 金融科技公司工程师的真实感受 · [English](cases/003-fintech-doc-pipeline/case.md) |
| [004-bigtech-infra](cases/004-bigtech-infra/case_zh.md) | 刚从小公司到大公司的工程师的感受 · [English](cases/004-bigtech-infra/case.md) |
| [_template](cases/_template/case_zh.md) | 复制以新建案例 · [English](cases/_template/case.md) |

## 工作原则

- **先 case 和 notes** — 记录真实发生的事；用 notes 读懂 [openai/codex](https://github.com/openai/codex) 等仓库
- 模式在多个 case 里重复出现，再往 `skills/`、`workflows/` 里加
- Codex 是协作者——人工关卡等有了 skill/workflow 再写进去

## 状态

当前重点是 case 和读 repo 的 notes。`skills/`、`workflows/` 仅占位（只有 `_template/`），等真实模式浮现再加。