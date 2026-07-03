# Case: Fintech backend — doc-driven delivery pipeline

**English** | [中文](case_zh.md)

**Type:** Voice interview / field notes  
**Source:** Voice chat with the interviewee  
**Synthesis:** [notes/real-workflows.md](../../notes/real-workflows.md)  
**Related cases:** [002-biotech-swe](../002-biotech-swe/case.md) · [004-bigtech-infra](../004-bigtech-infra/case.md)

---

## Context

**Fintech backend (5 in SG, team mostly in China):** one engineer, multi-repo. **Codex integrated with DingTalk docs.**

## Workflow

```text
Requirements arrive
    → PM + dev meeting → Meeting notes (AI summary)
    → Requirement Doc A (AI transform, PM steer)
    → Tech doc (AI transform, dev steer)
    → Code generation (coding agent, dev steer)
    → Requirement Doc B (AI from code)
    → Compare Doc A vs Doc B (dev + PM steer)
```

Frontend (Android / iOS / Web) consumes **AI-generated interface docs** from backend.

![Fintech doc pipeline](assets/4e332c2f541e80a18e3731e5d3d7bdb4.png)

## Observations

- **Fintech regulations** shape what can be automated
- Coding agents **improved productivity** but did **not** reduce overall workload

## Takeaways for codex-labs

| Observation            | In the lab                              |
| ---------------------- | --------------------------------------- |
| PM/dev steer every hop | `workflows/` human gates                |
| Doc A vs Doc B compare | Future `workflows/doc-driven-delivery/` |
| Domain harness + evals | `skills/` + `experiments/`              |