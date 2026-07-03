# Case: Knowledge Engineering for AI-native Repositories

**English** | [中文](case_zh.md)

**Type:** Markdown experience summary · Note synthesis  
**Source:** Anne's Markdown summary of an OpenAI-team Codex tips share seen on Xiaohongshu, plus her own interpretation  
**Original link:** https://www.xiaohongshu.com/explore/6a3929fb0000000020038fdb?source=webshare&xhsshare=pc_web&xsec_token=CB0zyTNhZ5-qf6Zz-GvOJje0Jyh18zo28MgAfu1Lme0cI=&xsec_source=pc_share

## Prompt / Notes Produced

- **Input note:** Anne's full Markdown summary — two core points from the share, plus her own four-layer model and AI-first repository sketch.
- **Related notes:** [method/agents-md.md](../../notes/method/agents-md.md) _(what AGENTS.md constrains — this case says what it should not become)_ · [codex/architecture.md](../../notes/codex/architecture.md)

## Context

The meta-point of the share is one sentence:

> The key to AI projects is no longer **Prompt Engineering**, but **Knowledge Engineering**.

A good AI project is not good because the prompt is beautifully written, but because **the whole repository is friendly enough for the agent**. Two concrete principles support this:

### 1. AI controls execution; humans control knowledge

| AI owns (execution layer)           | Humans own (knowledge layer)           |
| ----------------------------------- | -------------------------------------- |
| shell commands, code edits          | `AGENTS.md`, `docs/`                   |
| web search, tool calls              | skills, subagents                      |
| sandboxed execution, context compression | repository structure, coding conventions, architecture |

The right column determines how the AI understands the whole project — and only humans can maintain it over time.

### 2. Progressive Disclosure, not a 3,000-line AGENTS.md

Anti-pattern: putting the project intro, architecture, database, API, frontend, backend, tests, deployment, and style rules all into one `AGENTS.md`, until it grows to two or three thousand lines.

The better shape: `AGENTS.md` is a **directory** (~100–300 lines) that routes to focused documents:

```text
AGENTS.md                    docs/
├── Project Overview         ├── architecture.md
├── Repository Structure     ├── backend.md · frontend.md
└── Where to Find Things ──► ├── testing.md · deployment.md
                             └── api.md
```

Then each task loads only the knowledge it really needs:

```text
"Implement login" → read AGENTS.md → Authentication → docs/authentication.md → implement
```

instead of `Load Everything`. The result: smaller context, lower cost, higher accuracy, and easier maintenance.

## Interpretation Beyond the Original Share

| Layer                  | For    | Contents                                                                                | Question it answers              |
| ---------------------- | ------ | --------------------------------------------------------------------------------------- | -------------------------------- |
| 1 · Project structure  | Humans | `src/ tests/ scripts/ docs/`                                                            | Is the repo maintainable?        |
| 2 · Knowledge layer    | AI     | `AGENTS.md` (navigation only) + `docs/` + skills + subagents                            | **What does the AI know?**       |
| 3 · Workflow / Harness | AI     | `/implement` `/review` `/refactor` `/debug` — each picks skills, subagents, MCP, models | **How does the AI work?**        |
| 4 · Feedback loop      | System | generate → verify → evaluate → improve workflow → update skill                          | Does the system improve over time? |

Layer 4 is the most important future layer: workflows keep improving, skills keep updating, repository knowledge becomes more complete — a real closed loop.

Two additions from the comments:

- _"Harness Engineering is extremely important; it is what really turns a toy project into an engineering project."_
- One practice: **automatically scan the project directory before and after each Codex run, then update the structure docs** — knowledge is maintained automatically instead of manually.

## Target Shape of an AI-first Repository

```text
repo/
├── AGENTS.md        # navigation only (100–300 lines)
├── docs/            # long-lived knowledge
├── skills/          # reusable capabilities
├── workflows/       # task flows (feature / bugfix / review / release)
├── templates/       # issue / pr / adr
├── evals/           # verify whether AI actually did well
├── scripts/         # automatically maintain repo knowledge (e.g. update_project_map)
├── src/  └── tests/
```

A repository is no longer just a code repository — it is also the AI's **knowledge base, operating manual, and workflow system**.

## Takeaways

```text
Old model:
  better prompt → better answer

AI-first repository model:
  better repository knowledge
      → better agent context
      → better execution
      → better feedback
      → better repository knowledge (loop)
```

**Pattern:** Make `AGENTS.md` a router, not a warehouse. Put long-lived knowledge in focused documents; put reusable behavior in skills/workflows; put quality checks in evals; automate maintenance with scripts. The agent progressively discloses the context needed for the current task.

The best engineering teams of the future will not compete on model capability alone, but on who can build a better **AI-Native Engineering System**.
