# Testing

**English** | [中文](README_zh.md)

Plan tests, extend coverage, interpret failures, stabilize flaky tests.

## Outputs

- What to test and at which level (unit / integration / e2e)
- Concrete test cases or commands to run
- Interpretation of failures (regression vs env vs flake)

## Human checks

- Tests Codex added actually assert behavior, not implementation trivia
- CI commands match project defaults (`just test`, not raw `cargo test` unless intended)

## Related prompts

[`prompts/testing.md`](../../prompts/testing.md) _(add when ready)_

## Used in workflows

- [`workflows/new-project/`](../../workflows/new-project/)
- [`workflows/fix-bug/`](../../workflows/fix-bug/)
- [`workflows/implement-feature/`](../../workflows/implement-feature/)
- [`workflows/release/`](../../workflows/release/)