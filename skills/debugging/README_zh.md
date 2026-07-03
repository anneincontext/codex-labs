# Debugging

[English](README.md) | **中文**

缩小复现范围、读日志/堆栈、推断根因、提出最小修复。

## 产出

- 复现步骤（或证明是环境相关问题）
- 有证据支撑的根因假设
- 最小修复方案或下一步实验

## 人工核对

- Codex 是否在正确的 cwd 下跑了正确的命令？
- 修复是否被测试验证（失败变绿），而不只是「看起来对」？

## 文件

- [SKILL.md](SKILL.md) — 操作步骤
- [checklist.md](checklist.md) — 人工核对
- [prompts/debugging.md](../../prompts/debugging.md) / [debugging_zh.md](../../prompts/debugging_zh.md) — 触发模板

## 出现的 workflows

- [`workflows/fix-bug/`](../../workflows/fix-bug/)