# Codex Labs

[English](README.md) | **中文**

我和 Codex 共用的学习仓库——技能、流程、案例与笔记。

## 项目结构

```text
codex-labs/
├── skills/
│   ├── debugging/
│   ├── code-review/
│   ├── repo-understanding/
│   ├── documentation/
│   ├── architecture/
│   └── testing/
├── workflows/
│   ├── new-project/
│   ├── fix-bug/
│   ├── implement-feature/
│   └── release/
├── prompts/
├── cases/
│   ├── _template/
│   ├── 001-realtime-ai-presenter/
│   └── …
├── experiments/
├── notes/
└── resources/
```

## 如何串起来

```text
prompts  ──触发──►  skills  ──组合──►  workflows
cases / experiments ◄────────── 记录真实运行
notes / resources   ─────────── 原理与参考资料
```

## Agent 约束

| 文档 | English | 中文 |
| ---- | ------- | ---- |
| 本 lab 工作约定 | [AGENTS.md](AGENTS.md) | [AGENTS_zh.md](AGENTS_zh.md) |
| AGENTS.md 约束什么 | [notes/agents-md.md](notes/agents-md.md) | [notes/agents-md_cn.md](notes/agents-md_cn.md) |

## 从这里开始

| 主题 | English | 中文 |
| ---- | ------- | ---- |
| Skills 索引 | [skills/README.md](skills/README.md) | [skills/README_zh.md](skills/README_zh.md) |
| Workflows 索引 | [workflows/README.md](workflows/README.md) | [workflows/README_zh.md](workflows/README_zh.md) |
| Codex 架构 | [notes/architecture.md](notes/architecture.md) | [notes/architecture_cn.md](notes/architecture_cn.md) |
| 精选链接 | [resources/links.md](resources/links.md) | [resources/links_zh.md](resources/links_zh.md) |
| 案例索引 | [cases/README.md](cases/README.md) | [cases/README_zh.md](cases/README_zh.md) |
| 案例模板 | [cases/_template/case.md](cases/_template/case.md) | [cases/_template/case_zh.md](cases/_template/case_zh.md) |
| 田野调研（三次语音访谈） | [notes/real-workflows.md](notes/real-workflows.md) | [notes/real-workflows_cn.md](notes/real-workflows_cn.md) |

## Workflow 一览

| Workflow | 技能链 |
| -------- | ------ |
| [new-project](workflows/new-project/) | repo-understanding → architecture → documentation → testing |
| [fix-bug](workflows/fix-bug/) | debugging → testing → code-review |
| [implement-feature](workflows/implement-feature/) | repo-understanding → architecture → 实现 → testing → code-review |
| [release](workflows/release/) | testing → documentation → code-review |

## 工作原则

- Skill 要原子化；Workflow 组合 skill；Case 验证是否真的好用
- Codex 是协作者——人工关卡写在 skill/workflow 里
- 失败也要记录；最能教会你 workflow

## 状态

刚刚起步。每走通一条路径，就补对应的 `SKILL.md` 和 prompt 文件。