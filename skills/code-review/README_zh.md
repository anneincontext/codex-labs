# Code Review

[English](README.md) | **中文**

读 diff，关注正确性、安全、可维护性和是否可合并。

## 产出

- 分级意见（blocker / suggestion / nit）
- 需要作者澄清的问题
- 合并建议（含保留意见）

## 人工核对

- 安全与权限相关改动由你亲自把关
- 大重构：关键路径自己 spot-check

## 相关 prompts

[`prompts/code-review.md`](../../prompts/code-review.md) _(待补充)_

## 出现的 workflows

- [`workflows/fix-bug/`](../../workflows/fix-bug/)
- [`workflows/implement-feature/`](../../workflows/implement-feature/)
- [`workflows/release/`](../../workflows/release/)