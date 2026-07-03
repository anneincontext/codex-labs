# Testing

[English](README.md) | **中文**

规划测试、补覆盖、解读失败、处理不稳定测试。

## 产出

- 测什么、什么层级（单元 / 集成 / e2e）
- 具体用例或要跑的命令
- 失败解读（回归 vs 环境 vs flake）

## 人工核对

- 新增测试是否断言行为，而非实现细节
- CI 命令是否符合项目约定（如 `just test`）

## 相关 prompts

[`prompts/testing.md`](../../prompts/testing.md) _(待补充)_

## 出现的 workflows

- [`workflows/new-project/`](../../workflows/new-project/)
- [`workflows/fix-bug/`](../../workflows/fix-bug/)
- [`workflows/implement-feature/`](../../workflows/implement-feature/)
- [`workflows/release/`](../../workflows/release/)