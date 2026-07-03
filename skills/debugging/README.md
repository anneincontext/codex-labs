# Debugging

**English** | [中文](README_zh.md)

Narrow reproduction, read logs/stack traces, hypothesize root cause, propose minimal fixes.

## Outputs

- Repro steps (or proof it is environment-specific)
- Root-cause hypothesis with evidence
- Smallest fix or next experiment

## Human checks

- Did Codex run the right command in the right cwd?
- Is the fix verified by a failing test turning green (not just “looks right”)?

## Files

- [SKILL.md](SKILL.md) — procedure
- [checklist.md](checklist.md) — human gates
- [prompts/debugging.md](../../prompts/debugging.md) — trigger template

## Used in workflows

- [`workflows/fix-bug/`](../../workflows/fix-bug/)