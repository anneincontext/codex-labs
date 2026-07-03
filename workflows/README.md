# Workflows

**English** | [中文](README_zh.md)

**Multi-skill pipelines** — end-to-end processes by chaining [`skills/`](../skills/).

```text
workflows/
├── new-project/       # understand repo → document → baseline tests
├── fix-bug/           # debug → fix → test → review
├── implement-feature/ # understand → design → implement → test → review
└── release/           # test → docs → review → ship checklist
```

| Workflow | Folder | Status |
| -------- | ------ | ------ |
| New project | [new-project/](new-project/) | README only |
| Fix bug | [fix-bug/](fix-bug/) | README + [WORKFLOW.md](fix-bug/WORKFLOW.md) |
| Implement feature | [implement-feature/](implement-feature/) | README only |
| Release | [release/](release/) | README only |

Copy [_template/](_template/) for `WORKFLOW.md`. Link skills; don't duplicate skill text.