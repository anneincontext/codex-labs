# Codex Labs — 工作约定

维护本仓库时，Codex（和人）应遵守的规则。

## 始终

- **skills** 保持原子能力；多步流程只放在 **workflows**。
- 每个新目录提供双语：`README.md` + `README_zh.md`。
- 指向 [openai/codex](https://github.com/openai/codex) 的链接写在 `resources/`，使用完整 GitHub URL。
- 某条 workflow 证明有用（或失败得很有教训）时，在 `cases/` 记一篇复盘。

## 新增内容前

- 优先扩展现有 skill/workflow，避免近 Duplicate 目录。
- 新 skill → 必须有对应 `prompts/<name>.md`（可先写 stub）。
- 新外部项目引用 → 同时更新 `resources/links.md` 与 `links_zh.md`。

## 文件约定

| 位置 | 必备文件 |
| ---- | -------- |
| `skills/<name>/` | `README.md`、`README_zh.md`、`SKILL.md`、`checklist.md` |
| `workflows/<name>/` | `README.md`、`README_zh.md`、`WORKFLOW.md` |
| `prompts/` | 触发该 skill 的 `<skill-name>.md` |

## 原则

优先：短小可复用、低决策成本、具体案例胜过空泛建议。

避免：在本 lab 重复 openai/codex 官方文档、巨型单文件、名 skill 实 workflow。