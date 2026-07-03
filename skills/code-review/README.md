# Code Review

**English** | [中文](README_zh.md)

Read diffs for correctness, security, maintainability, and merge readiness.

## Outputs

- Ordered findings (blocker / suggestion / nit)
- Questions for the author
- Merge recommendation (with caveats)

## Human checks

- Security and permission changes reviewed by you
- Large refactors: spot-check critical paths yourself

## Related prompts

[`prompts/code-review.md`](../../prompts/code-review.md) _(add when ready)_

## Used in workflows

- [`workflows/fix-bug/`](../../workflows/fix-bug/)
- [`workflows/implement-feature/`](../../workflows/implement-feature/)
- [`workflows/release/`](../../workflows/release/)