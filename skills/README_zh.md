# Skills

[English](README.md) | **中文**

可复用的**能力**——Codex（和你）能稳定做好的原子技能。

```text
skills/
├── debugging/           # 缩小复现、日志、根因、修复建议
├── code-review/         # 读 diff、风险、风格、是否可合并
├── repo-understanding/  # 摸清结构、入口、约定
├── documentation/       # README、指南、清晰文字
├── architecture/        # 系统设计笔记、图示、权衡
└── testing/             # 测试计划、覆盖缺口、不稳定测试
```

| 技能 | 目录 | 状态 |
| ---- | ---- | ---- |
| Debugging | [debugging/](debugging/) | 已有 SKILL + checklist + [prompt](../prompts/debugging.md) |
| Code review | [code-review/](code-review/) | 仅 README |
| Repo understanding | [repo-understanding/](repo-understanding/) | 仅 README |
| Documentation | [documentation/](documentation/) | 仅 README |
| Architecture | [architecture/](architecture/) | 仅 README |
| Testing | [testing/](testing/) | 仅 README |

新 skill 复制 [_template/](_template/)：`SKILL.md` + `checklist.md`。

## 与其他目录的关系

```text
prompts/     ──触发──►  skills/
skills/      ──组合──►  workflows/
cases/       ──记录──►  真实运行
notes/       ──解释──►  技能背后的原理
```