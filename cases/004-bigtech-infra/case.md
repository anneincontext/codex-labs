# Case: Big Tech infra — domain harness & maturity ladder

**English** | [中文](case_zh.md)

**Type:** Voice interview / field notes  
**Source:** Voice chat with the interviewee  
**Synthesis:** [notes/real-workflows.md](../../notes/real-workflows.md)  
**Related cases:** [002-biotech-swe](../002-biotech-swe/case.md) · [003-fintech-doc-pipeline](../003-fintech-doc-pipeline/case.md)

---

## Context

**Big Tech infra engineer:** joined recently; architecture + business **tech debt** (small company → big company). Messaging / async systems. In infra, **the engineer is the product**. ~$1000/mo tooling + Cursor Enterprise.

## Harness / pipeline view

From a prior small company: **software production as a pipeline** — practicing **harness engineering** in a **domain-specific** way:

- Strong **test platform**
- Strong **observability** (middleware state)
- Cannot discuss harness without the **business domain**
- Both **code harness** and **workflow harness** are domain-tied

Org bought several shared OpenAI accounts (relay) for team access.

## AI usage

- **Ops:** cluster ops, alert governance, incident response
- **R&D:** code generation
- **Pipeline built incrementally** with **domain benchmarks / evals**
- Tension: **feature pressure** vs **system optimization**

## Maturity model (their framing)

Not a clean upgrade path — **hand-written code still happens today**, alongside the later layers:

```text
1. Hand-written code        — still common in daily work
2. Human-in-the-loop        — harness insufficient; agent lacks env signals for next step
3. Multi-agent              — software engineer defines goals, designs pipeline
```

Onboarding: **checklist for newcomers**.

## Personal stack

- OpenClaw, **codex-cli**
- Claude models blocked domestically → routing workarounds
- Codex: chat or **Codex SDK** for self-managed loops

**Works well for them:** large codebase interpretation (e.g. clone codex repo and ask it to explain), interview recap with AI

## Takeaways for codex-labs

| Observation | Where it lives in the lab |
| ----------- | ------------------------ |
| Domain harness + evals | `skills/` + `experiments/` per domain |
| Anne's 20 min demo | [001-realtime-ai-presenter](../001-realtime-ai-presenter/case.md) |