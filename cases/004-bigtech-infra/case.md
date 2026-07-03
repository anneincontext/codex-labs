# Case: Big Tech infra — domain harness & maturity ladder

**English** | [中文](case_zh.md)

**Type:** Voice interview / field notes  
**Source:** Voice chat with the interviewee  
**Synthesis:** [notes/research/real-workflows.md](../../notes/research/real-workflows.md)  
**Related cases:** [002-biotech-swe](../002-biotech-swe/case.md) · [003-fintech-doc-pipeline](../003-fintech-doc-pipeline/case.md)

---

## Context

**Big Tech infra engineer:** moved from a **small company to a big one** recently — **just joined, has not started on production code yet**. _The engineer is the product_ is how he frames the infra role, not a report of what he is doing at the big company today.

**Company AI allowance:** about **$1,000/month** in tooling budget plus **Cursor Enterprise** (company-paid).

## Small company vs big company (their contrast)

| Dimension       | Small company                                              | Big company                                              |
| --------------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| **Tech debt**   | Lighter; cheaper to change course                          | **More** — architecture and business debt stacked        |
| **Workflow & AI** | **More AI-embracing** — designs production as a pipeline | More inertia; feature delivery beats workflow redesign   |

One extra point (**not big-company-specific**): **harness gets built incrementally everywhere** — no team gets it in one shot.

Overall impression from the chat: **tools are not the bottleneck** (the allowance is real). In a big company, **heavier tech debt and faster pace** make it **harder to embrace AI in the workflow** the way a small company can — that is his view **entering** big co., drawn from prior experience, not a post-onboarding writeup.

## Harness / pipeline view (small-company experience)

Mostly from **his small-company years**, not current big-company practice:

- **Software production as a pipeline** — harness engineering in a **domain-specific** way
- Test platform, observability, middleware state matter
- Cannot discuss harness without the **business domain**
- Both **code harness** and **workflow harness** are domain-tied

## Maturity model (their framing)

Not a clean upgrade path — **hand-written code still happens today**, alongside the later layers:

```text
1. Hand-written code        — still common in daily work
2. Human-in-the-loop        — harness insufficient; agent lacks env signals for next step
3. Multi-agent              — software engineer defines goals, designs pipeline
```

## Personal stack

Personal tools — not tied to big-company work yet:

- OpenClaw, **codex-cli**
- Claude models blocked domestically → routing workarounds
- Codex: chat or **Codex SDK** — he said it helps with **large-repo / code interpretation**, interview recap, etc.

That thread later became concrete: Anne used the same approach on [openai/codex](https://github.com/openai/codex), and **[codex-labs exists because of it](../005-learn-codex-repo/case.md)**. His instinct checks out — **AI is genuinely effective at helping people understand a codebase**, with a much lower entry cost than reading source cold.

## Takeaways for codex-labs

| Observation             | In the lab                                               |
| ----------------------- | -------------------------------------------------------- |
| Codebase interpretation | [005-learn-codex-repo](../005-learn-codex-repo/case.md) — why this lab exists |
| Domain harness + evals  | `skills/` + `experiments/`                               |
