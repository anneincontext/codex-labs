# Workflows

[English](README.md) | **中文**

**多技能组合的完整流程**——串联 [`skills/`](../skills/) 完成端到端任务。

```text
workflows/
├── new-project/       # 理解仓库 → 文档化 → 基线测试
├── fix-bug/           # 调试 → 修复 → 测试 → review
├── implement-feature/ # 理解 → 设计 → 实现 → 测试 → review
└── release/           # 测试 → 文档 → review → 发布清单
```

| Workflow | 目录 | 状态 |
| -------- | ---- | ---- |
| New project | [new-project/](new-project/) | 仅 README |
| Fix bug | [fix-bug/](fix-bug/) | README + [WORKFLOW.md](fix-bug/WORKFLOW.md) |
| Implement feature | [implement-feature/](implement-feature/) | 仅 README |
| Release | [release/](release/) | 仅 README |

新 workflow 复制 [_template/](_template/) 写 `WORKFLOW.md`。链接 skill，不重复 skill 正文。