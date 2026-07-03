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

| 位置                | 必备文件                                                    |
| ------------------- | ----------------------------------------------------------- |
| `skills/<name>/`    | `README.md`、`README_zh.md`、`SKILL.md`、`checklist.md`     |
| `workflows/<name>/` | `README.md`、`README_zh.md`、`WORKFLOW.md`                  |
| `prompts/`          | 触发该 skill 的 `<skill-name>.md`                           |

## 表格与图示

**写完先对齐再看一遍** — 表格不齐是本仓库常见的返工原因之一。

### Markdown 表格

- 每一列都要有**表头**（不要 `| 案例 | |` 这种空列名）。
- 分隔行的 `-` **与表头列宽一致**。
- 单元格 padding 让闭合的 `|` 在**等宽字体**里纵向对齐。
- **中文 / 中英混排：** 按**显示宽度** padding（CJK 计 2 格，ASCII 计 1 格），不是按字符个数。
- 简介放第二列；语言切换用行尾的 `· [中文]` / `· [English]`，省一列。

### 流程图 — 用 [D2](https://d2lang.com/)

- **不要**手写 ASCII 框图做多步流程 — 尤其中文很难对齐。
- 源文件放 `notes/diagrams/<topic>/`，扩展名 `.d2`；同时提交 `.svg` 方便 GitHub 预览。
- 渲染：`d2 --layout dagre file.d2 file.svg`（见 [diagrams/learn-codebase-flow/README.md](notes/diagrams/learn-codebase-flow/README.md)）。
- 双语：英文 `foo.d2` + `foo.svg`，中文 `foo_cn.d2` + `foo_cn.svg`（标签不同时）。
- Markdown 嵌入：`![说明](diagrams/.../foo.svg)`，并附 **图源：** 链到 `.d2`。
- 示例：[notes/learn-codebase-flow_cn.md](notes/learn-codebase-flow_cn.md)。

### 提交前

- 在等宽编辑器里检查表格右边界。
- 改过 `.d2` 后重新渲染，并把 `.svg` 一并提交。

## 原则

优先：短小可复用、低决策成本、具体案例胜过空泛建议。

避免：在本 lab 重复 openai/codex 官方文档、巨型单文件、名 skill 实 workflow。