# Codex Labs

**English** | [中文](README_zh.md)

A shared canvas for me and Codex — skills, workflows, cases, and notes from real use.

## Structure

```text
codex-labs/
├── skills/          # _template/ only — add skills from real cases
├── workflows/       # _template/ only — add when skills chain
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

### Notes

| Topic | English | 中文 |
| ----- | ------- | ---- |
| Codex architecture | [notes/architecture.md](notes/architecture.md) | [notes/architecture_cn.md](notes/architecture_cn.md) |
| Layered design | [notes/layeredDesign.md](notes/layeredDesign.md) | [notes/layeredDesign_cn.md](notes/layeredDesign_cn.md) |
| What AGENTS.md constrains | [notes/agents-md.md](notes/agents-md.md) | [notes/agents-md_cn.md](notes/agents-md_cn.md) |
| Field research (3 voice interviews) | [notes/real-workflows.md](notes/real-workflows.md) | [notes/real-workflows_cn.md](notes/real-workflows_cn.md) |

### Cases

| Case | |
| ---- | --- |
| [005-learn-codex-repo](cases/005-learn-codex-repo/case.md) | Why this repo exists — learning a codebase with Codex · [中文](cases/005-learn-codex-repo/case_zh.md) |
| [001-realtime-ai-presenter](cases/001-realtime-ai-presenter/case.md) | Building an OpenAI Realtime API demo with Codex · [中文](cases/001-realtime-ai-presenter/case_zh.md) |
| [002-biotech-swe](cases/002-biotech-swe/case.md) | A biotech engineer's real experience · [中文](cases/002-biotech-swe/case_zh.md) |
| [003-fintech-doc-pipeline](cases/003-fintech-doc-pipeline/case.md) | A fintech engineer's real experience · [中文](cases/003-fintech-doc-pipeline/case_zh.md) |
| [004-bigtech-infra](cases/004-bigtech-infra/case.md) | An engineer who just moved from a small company to a big one · [中文](cases/004-bigtech-infra/case_zh.md) |
| [_template](cases/_template/case.md) | Copy to start a new case · [中文](cases/_template/case_zh.md) |

## Working principles

- **Cases and notes first** — record what happened; use notes to understand repos like [openai/codex](https://github.com/openai/codex)
- Add skills and workflows only when a pattern repeats across cases
- Treat Codex as a collaborator — human gates belong in skill/workflow docs when you add them

## Status

Cases and repo-reading notes are the focus. `skills/` and `workflows/` are placeholders (`_template/` only) until real patterns emerge.