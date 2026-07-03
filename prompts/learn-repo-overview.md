# Prompt: Learn a codebase — high-level overview

**English** | [中文](learn-repo-overview_zh.md)

**Case:** [cases/005-learn-codex-repo/case.md](../cases/005-learn-codex-repo/case.md)  
**Flow (prompt → result):** [notes/learn-codebase-flow.md](../notes/learn-codebase-flow.md)  
**Target repo (example):** [openai/codex](https://github.com/openai/codex)

First prompt Anne used to learn a large unfamiliar repo with Codex. Works on any codebase you can open in the agent's workspace.

---

## Template

```text
Give me a high-level overview of this repository.

Please explain:
- what this project does
- the main architecture
- important folders
- how requests flow through the system
- what technologies are used
```

---

## Why these five bullets

| Question          | What you get                                       |
| ----------------- | -------------------------------------------------- |
| What it does      | Purpose and boundaries — in / out of scope         |
| Main architecture | Mental model before file-level reading           |
| Important folders | Where to look next; avoids random `grep`           |
| Request flow      | How runtime behavior connects modules              |
| Technologies      | Stack, build system, dependencies                  |

You are asking for a **map**, not a line-by-line tour. Follow-up turns can zoom into one layer, one crate, or one path.

---

## Suggested follow-ups (after the overview)

```text
Draw a Mermaid diagram of the request flow and align it with folder names.

Walk through one user turn end-to-end — which crates and files are involved?

What would you read first if you needed to change [X]?
```

Capture good answers in `notes/` (this lab) or the target repo's docs if you own it.