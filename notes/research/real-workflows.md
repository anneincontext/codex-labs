# Real-world AI workflows (field research)

**English** | [中文](real-workflows_zh.md)

> **Answers:** how practitioners actually use AI coding tools (3 interviews, synthesized).
> **Read first:** nothing; per-person detail lives in [cases/](../../cases/).

Synthesis from three voice interviews with practitioners. Full writeups: [002-biotech-swe](../../cases/002-biotech-swe/case.md) · [003-fintech-doc-pipeline](../../cases/003-fintech-doc-pipeline/case.md) · [004-bigtech-infra](../../cases/004-bigtech-infra/case.md).

## Cross-cutting patterns

### 1. Human steer at every AI hop

None of the three treat AI as autopilot. Each transform step has a **role owner**:

| Step | Typical owner |
| ---- | ------------- |
| Requirements → ticket / doc | Product manager |
| Doc → tech spec | Developer |
| Spec → code | Developer |
| Code → verify / ship | Human CI/CD, test, MR |

**Lab mapping:** `skills/*/checklist.md` + `workflows/*/WORKFLOW.md` human gates.

### 2. Adoption is often awareness, not tooling

Biotech example: engineers still resolve Git conflicts manually because they **did not know** agents could do it.

**Implication for codex-labs:** document *what agents can do* (skills index), not only how to prompt.

### 3. Docs as contracts (Fintech)

Req Doc A (pre-code) vs Req Doc B (derived from code) → **compare** under PM + dev steer. AI moves text; humans own alignment.

**Lab mapping:** future workflow `doc-driven-delivery/`; skill `documentation/`.

### 4. Small cos embrace AI workflows; big cos carry more debt (infra engineer)

Interviewee moved from a **small company to a big one** — **just joined, not on production code yet**. Company AI allowance is real (~**$1k/mo** tooling + **Cursor Enterprise**). Contrast: **more tech debt** at big co., **less AI-embracing workflows** than his small-co. experience; **harness is built incrementally everywhere**, not a big-co.-only problem.

Harness views mostly come from small-company years; still domain-tied (test platform, observability, middleware state, etc.).

He also said Codex works well for **codebase interpretation** — Anne later used the same approach on openai/codex, which is [why codex-labs exists](../../cases/005-learn-codex-repo/case.md); AI is genuinely effective at helping people understand a repo.

**Lab mapping:** [agents-md.md](../method/agents-md.md) is project-local; org harness = per-domain skills + evals. Full case: [004-bigtech-infra](../../cases/004-bigtech-infra/case.md).

### 5. Productivity ≠ less work

Fintech: coding agents improved throughput; **total load did not shrink** (regulation, review, doc chain).

### 6. Maturity ladder (infra engineer)

Not a clean upgrade path — **hand-written code still happens today**, alongside the later layers:

```text
Hand-written code (still common day to day)
    → Human-in-the-loop (harness not enough; env lacks next-step signal)
        → Multi-agent (engineer defines goals, designs the pipeline)
```

**Lab mapping:** `workflows/` = designed pipelines; `AGENTS.md` = loop constraints.

## Constraints seen in the wild

| Constraint | Where |
| ---------- | ----- |
| Confidential / gene data | Biotech |
| Fintech regulation | Fintech |
| Enterprise tool limits ($120 Cursor not enough) | Biotech |
| Model access (Claude blocked, shared relay accounts) | Big Tech / CN |

## Tooling mentioned

- Cursor enterprise, personal subscriptions on top
- Codex + DingTalk (DingDing) docs
- Global `AGENTS.md`
- codex-cli, OpenClaw

## Cases

- [002-biotech-swe](../../cases/002-biotech-swe/case.md) · [003-fintech-doc-pipeline](../../cases/003-fintech-doc-pipeline/case.md) · [004-bigtech-infra](../../cases/004-bigtech-infra/case.md)