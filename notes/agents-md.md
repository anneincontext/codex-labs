# What does AGENTS.md constrain?

**English** | [中文](agents-md_cn.md)

`AGENTS.md` is a **project-local contract** injected into the agent's context. It does not run code — it **shapes decisions** before and during work.

## Example: [realtime-ai-presenter/AGENTS.md](https://github.com/anneincontext/realtime-ai-presenter/blob/main/AGENTS.md)

That file is short (~25 lines) but constrains four layers:

### 1. Working agreements (hard rules)

| Rule | What it constrains |
| ---- | ------------------ |
| Run `npm test` after modifying JavaScript | **Verification** — agent must not finish JS edits without running tests |
| Prefer `pnpm` for installs | **Tooling** — package manager choice |
| Ask before adding production dependencies | **Approval gate** — human consent for dependency growth |

These are closest to **executable policy**: specific, checkable, often mechanical.

### 2. Principles — prioritize

| Principle | What it constrains |
| --------- | ------------------ |
| Readable code | Style and naming — favor clarity over cleverness |
| Low agent decision cost | Architecture — fewer branches for the agent to guess |
| Small independent modules | Structure — file/module granularity |
| Future extensibility | API shape — leave seams, avoid dead ends |
| Developer friendliness | DX — docs, errors, predictable layout |

These guide **tradeoffs** when multiple valid implementations exist.

### 3. Principles — avoid

| Anti-pattern | What it constrains |
| ------------ | ------------------ |
| Large abstractions | Don't introduce frameworks/wrappers without need |
| Complicated state management | Prefer local/simple state |
| Premature optimization | Don't optimize before correctness/clarity |
| Heavy dependencies | Weight of `package.json` and supply chain |

These are **negative constraints** — things the agent should not reach for by default.

### 4. Change strategy

| Rule | What it constrains |
| ---- | ------------------ |
| Extend existing code over rewriting | **Edit scope** — patch and grow, don't replace wholesale |
| Simple, boring solutions | **Solution class** — reject flashy designs |

## Mental model

```text
AGENTS.md
├── Working agreements   →  checkable habits (test, pnpm, ask first)
├── Priorities           →  tie-breakers when designing
├── Avoid list           →  default "no" list
└── Change strategy      →  how to touch the codebase
```

Compare with [openai/codex AGENTS.md](https://github.com/openai/codex/blob/main/AGENTS.md): same idea, much larger — crate rules, sandbox env vars, test commands, Clippy, app-server API shape, etc.

## codex-labs mapping

| AGENTS.md layer | Where it lives here |
| --------------- | ------------------- |
| Repo-wide rules | [`AGENTS.md`](../AGENTS.md) |
| Skill procedure | `skills/<name>/SKILL.md` |
| Human gates | `skills/<name>/checklist.md` |
| Multi-skill flow | `workflows/<name>/WORKFLOW.md` |
| Trigger template | `prompts/<name>.md` |

Your presenter demo shows that **a small AGENTS.md beats a long system prompt** when the constraints are stable and project-specific.

## prompt.md vs AGENTS.md (Realtime case)

Anne used [`Realtime api prompt`](../prompts/implement-demo.md) to **bootstrap** the demo and [AGENTS.md](https://github.com/anneincontext/realtime-ai-presenter/blob/main/AGENTS.md) to **maintain** it:

| File | When | Constrains |
| ---- | ---- | ---------- |
| `prompt.md` | First session (or big feature) | Product goal, base repo, success criteria, verification, scope caps |
| `AGENTS.md` | Every later edit | Test habits, toolchain, dep approval, taste, anti-patterns |

Case writeup: [cases/001-realtime-ai-presenter.md](../cases/001-realtime-ai-presenter.md).