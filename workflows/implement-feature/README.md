# Implement Feature

**English** | [中文](README_zh.md)

Design and land a scoped feature change.

## Skill sequence

```text
repo-understanding → architecture → (implement) → testing → code-review
```

| Step | Skill | Goal |
| ---- | ----- | ---- |
| 1 | [repo-understanding](../../skills/repo-understanding/) | Touch only the right modules |
| 2 | [architecture](../../skills/architecture/) | Where the feature fits; API shape |
| 3 | _implementation_ | Codex edits + your steering |
| 4 | [testing](../../skills/testing/) | New tests + existing suite green |
| 5 | [code-review](../../skills/code-review/) | Review full diff before PR |

## Human gates

- After architecture: you agree with the seam and public API
- Before PR: scope matches the original ask (no drive-by refactors)

## Done when

Feature works, tests cover main paths, diff reviewed.