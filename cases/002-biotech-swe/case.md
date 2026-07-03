# Case: Biotech SWE — JIRA to Cursor pipeline

**English** | [中文](case_zh.md)

**Type:** Voice interview / field notes  
**Source:** Voice chat with the interviewee  
**Synthesis:** [notes/real-workflows.md](../../notes/real-workflows.md)  
**Related cases:** [003-fintech-doc-pipeline](../003-fintech-doc-pipeline/case.md) · [004-bigtech-infra](../004-bigtech-infra/case.md)

---

## Context

**Biotech SWE team (~30 people):** one scrum team, multi-repo. Cursor Enterprise (~$120/person) — **not enough**; engineer also uses a personal subscription to finish work.

## Workflow

```text
Requirements → JIRA ticket (AI transform, PM steer)
           → Cursor code generation
           → Human: CI/CD / test / merge request
```

Global **`AGENTS.md`** instructs the AI org-wide.

![Biotech workflow](assets/c4d828cb4b228b1c630bf5fdd3612f5a.png)

## Constraints & attitude

- **Confidential** gene-related data limits what can be fed to models
- Team **still getting used to** AI

## Notable quote

Two colleagues resolve Git conflicts **manually** — they had not realized an agent could do it.  
→ **AI adoption is often an awareness problem**, not a tooling problem.

## Takeaways for codex-labs

| Observation          | In the lab                                               |
| -------------------- | -------------------------------------------------------- |
| PM steer at each hop | `workflows/` human gates                                 |
| Global AGENTS.md     | [AGENTS.md](../../AGENTS.md), [agents-md.md](../../notes/agents-md.md) |
| Awareness gap        | [notes/real-workflows.md](../../notes/real-workflows.md) |