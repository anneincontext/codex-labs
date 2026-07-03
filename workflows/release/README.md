# Release

**English** | [中文](README_zh.md)

Prepare a change set for shipping: quality bar, docs, final review.

## Skill sequence

```text
testing → documentation → code-review
```

| Step | Skill | Goal |
| ---- | ----- | ---- |
| 1 | [testing](../../skills/testing/) | Full relevant suite; no known flakes |
| 2 | [documentation](../../skills/documentation/) | Changelog, README, migration notes if needed |
| 3 | [code-review](../../skills/code-review/) | Final pass on release diff |

## Human gates

- Version numbers and breaking changes: your call
- Production deploy: never delegated to Codex alone

## Done when

You would ship to users today with a clear rollback story.