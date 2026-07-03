# Fix Bug

**English** | [中文](README_zh.md)

From failing behavior to a reviewed, tested fix.

## Skill sequence

```text
debugging → testing → code-review
```

| Step | Skill | Goal |
| ---- | ----- | ---- |
| 1 | [debugging](../../skills/debugging/) | Repro, root cause, minimal fix |
| 2 | [testing](../../skills/testing/) | Regression test; green suite |
| 3 | [code-review](../../skills/code-review/) | Self-review or Codex review of diff |

## Human gates

- Before merge: you ran tests locally
- Security-sensitive paths: you read the diff yourself

## Files

- [WORKFLOW.md](WORKFLOW.md) — full flow with prompt links

## Done when

Bug is fixed, regression test exists, and you would approve the PR.