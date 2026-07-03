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
└── resources/
```

## 如何串起来

```text
prompts  ──触发──►  skills  ──组合──►  workflows
cases / experiments ◄────────── 记录真实运行
notes / resources   ─────────── 原理与参考资料
```

## Agent 约束

| 文档 | English | 中文 |
| ---- | ------- | ---- |
| 本 lab 工作约定 | [AGENTS.md](AGENTS.md) | [AGENTS_zh.md](AGENTS_zh.md) |
| AGENTS.md 约束什么 | [notes/agents-md.md](notes/agents-md.md) | [notes/agents-md_cn.md](notes/agents-md_cn.md) |

## 从这里开始

### Notes

| 主题 | English | 中文 |
| ---- | ------- | ---- |
| Codex 架构 | [notes/architecture.md](notes/architecture.md) | [notes/architecture_cn.md](notes/architecture_cn.md) |
| 分层架构 | [notes/layeredDesign.md](notes/layeredDesign.md) | [notes/layeredDesign_cn.md](notes/layeredDesign_cn.md) |
| AGENTS.md 约束什么 | [notes/agents-md.md](notes/agents-md.md) | [notes/agents-md_cn.md](notes/agents-md_cn.md) |
| 田野调研（三次语音访谈） | [notes/real-workflows.md](notes/real-workflows.md) | [notes/real-workflows_cn.md](notes/real-workflows_cn.md) |

### 案例

| 案例 | English | 中文 |
| ---- | ------- | ---- |
| 005 — 用 AI 学习 openai/codex | [cases/005-learn-codex-repo/case.md](cases/005-learn-codex-repo/case.md) | [cases/005-learn-codex-repo/case_zh.md](cases/005-learn-codex-repo/case_zh.md) |
| 001 — Realtime AI Presenter demo | [cases/001-realtime-ai-presenter/case.md](cases/001-realtime-ai-presenter/case.md) | [cases/001-realtime-ai-presenter/case_zh.md](cases/001-realtime-ai-presenter/case_zh.md) |
| 002 — 生物科技 SWE | [cases/002-biotech-swe/case.md](cases/002-biotech-swe/case.md) | [cases/002-biotech-swe/case_zh.md](cases/002-biotech-swe/case_zh.md) |
| 003 — 金融科技文档流水线 | [cases/003-fintech-doc-pipeline/case.md](cases/003-fintech-doc-pipeline/case.md) | [cases/003-fintech-doc-pipeline/case_zh.md](cases/003-fintech-doc-pipeline/case_zh.md) |
| 004 — 大厂 infra | [cases/004-bigtech-infra/case.md](cases/004-bigtech-infra/case.md) | [cases/004-bigtech-infra/case_zh.md](cases/004-bigtech-infra/case_zh.md) |
| 模板 | [cases/_template/case.md](cases/_template/case.md) | [cases/_template/case_zh.md](cases/_template/case_zh.md) |

## 工作原则

- **先 case 和 notes** — 记录真实发生的事；用 notes 读懂 [openai/codex](https://github.com/openai/codex) 等仓库
- 模式在多个 case 里重复出现，再往 `skills/`、`workflows/` 里加
- Codex 是协作者——人工关卡等有了 skill/workflow 再写进去

## 状态

当前重点是 case 和读 repo 的 notes。`skills/`、`workflows/` 仅占位（只有 `_template/`），等真实模式浮现再加。