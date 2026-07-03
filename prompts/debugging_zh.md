# Prompt: Debugging

**Skill：** [skills/debugging/](../skills/debugging/)

## 模板

```text
在 {{project}} 里排查这个问题。

现象：{{symptom}}
复现（若已知）：{{repro}}

1. 用项目的测试/lint 命令确认复现（有 AGENTS.md 先读它）。
2. 给出带 file:line 证据的根因。
3. 提出最小修复。
4. 补充或更新回归测试。

不要未经询问添加依赖。
```

## 参数

- `{{project}}` — 仓库名或路径
- `{{symptom}}` — 异常表现
- `{{repro}}` — 步骤或命令