# Fix Bug

[English](README.md) | **中文**

从异常现象到经过 review、有测试保障的修复。

## 技能序列

```text
debugging → testing → code-review
```

| 步骤 | Skill | 目标 |
| ---- | ----- | ---- |
| 1 | [debugging](../../skills/debugging/) | 复现、根因、最小修复 |
| 2 | [testing](../../skills/testing/) | 回归测试；测试全绿 |
| 3 | [code-review](../../skills/code-review/) | 自审或让 Codex review diff |

## 人工关卡

- 合并前：你在本地跑过测试
- 安全敏感路径：你亲自读过 diff

## 文件

- [WORKFLOW.md](WORKFLOW.md) — 完整流程与 prompt 链接

## 完成标准

Bug 已修、有回归测试、你会批准这个 PR。