# Codex Labs

**English** | [中文](README_zh.md)

A shared canvas for me and Codex — skills, workflows, cases, and notes from real use.

## Structure

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

## How it fits together

```text
prompts  ──trigger──►  skills  ──combine──►  workflows
cases / experiments ◄────────── record real runs
notes / resources   ────────── theory & references
```

## Agent constraints

| Doc | English | 中文 |
| --- | ------- | ---- |
| Lab working agreements | [AGENTS.md](AGENTS.md) | [AGENTS_zh.md](AGENTS_zh.md) |
| What AGENTS.md constrains | [notes/agents-md.md](notes/agents-md.md) | [notes/agents-md_cn.md](notes/agents-md_cn.md) |

## Start here

| Topic | English | 中文 |
| ----- | ------- | ---- |
| Skills index | [skills/README.md](skills/README.md) | [skills/README_zh.md](skills/README_zh.md) |
| Workflows index | [workflows/README.md](workflows/README.md) | [workflows/README_zh.md](workflows/README_zh.md) |
| Codex architecture | [notes/architecture.md](notes/architecture.md) | [notes/architecture_cn.md](notes/architecture_cn.md) |
| Curated links | [resources/links.md](resources/links.md) | [resources/links_zh.md](resources/links_zh.md) |
| Cases index | [cases/README.md](cases/README.md) | [cases/README_zh.md](cases/README_zh.md) |
| Case template | [cases/_template/case.md](cases/_template/case.md) | [cases/_template/case_zh.md](cases/_template/case_zh.md) |
| Field research (3 voice interviews) | [notes/real-workflows.md](notes/real-workflows.md) | [notes/real-workflows_cn.md](notes/real-workflows_cn.md) |

## Workflows at a glance

| Workflow | Skills chain |
| -------- | ------------ |
| [new-project](workflows/new-project/) | repo-understanding → architecture → documentation → testing |
| [fix-bug](workflows/fix-bug/) | debugging → testing → code-review |
| [implement-feature](workflows/implement-feature/) | repo-understanding → architecture → implement → testing → code-review |
| [release](workflows/release/) | testing → documentation → code-review |

## Working principles

- Skills are atomic; workflows compose them; cases prove they work
- Treat Codex as a collaborator — human gates stay in skill/workflow docs
- Capture failures; they teach the best workflows

## Status

Just getting started. Add `SKILL.md` / prompt files as you use each path in anger.