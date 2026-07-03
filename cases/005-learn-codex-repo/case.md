# Case: Learn openai/codex with AI

**English** | [中文](case_zh.md)

**Type:** Own learning session (Anne + Codex)  
**Source:** Anne used Codex to read and learn the [openai/codex](https://github.com/openai/codex) repo  
**Target repo:** [openai/codex](https://github.com/openai/codex) (~100 Rust crates, CLI + TUI + app-server)

## Prompts / Notes produced

- **Flow (prompt → result):** [notes/method/learn-codebase-flow.md](../../notes/method/learn-codebase-flow.md) / [中文](../../notes/method/learn-codebase-flow_zh.md)
- **First prompt:** [prompts/learn-repo-overview.md](../../prompts/learn-repo-overview.md)
- **Architecture overview:** [notes/codex/architecture.md](../../notes/codex/architecture.md) / [中文](../../notes/codex/architecture_zh.md)
- **Layer-by-layer walkthrough:** [notes/codex/layered-design.md](../../notes/codex/layered-design.md) / [中文](../../notes/codex/layered-design_zh.md)

## Context

Codex is a large, unfamiliar codebase. Reading source directly — jumping between crates, build files, and protocol types — used to feel too heavy to start.

In [case 004](../004-bigtech-infra/case.md), the interviewee also said Codex works well for **large-repo / code interpretation**. Anne tried it and agreed — **AI is genuinely effective at helping people understand a codebase** — which is why **codex-labs exists**.

Instead: open the repo in Codex and ask for a structured overview first. Turn the agent into a **guided tour** before deep reading.

## What I Asked Codex

```text
Give me a high-level overview of this repository.

Please explain:
- what this project does
- the main architecture
- important folders
- how requests flow through the system
- what technologies are used
```

One prompt, five sections. No need to know entry points upfront — the questions define what "done" looks like for a first pass.

## What Codex Did

- Summarized what Codex CLI does (local agent, tools, threads, sandbox)
- Mapped `codex-rs/` crate layout: CLI, TUI, core, app-server, exec, protocol, …
- Explained request flow: user input → session → model → tools → sandbox → events back to UI
- Listed technologies: Rust workspace, Bazel/Cargo, TypeScript SDK packaging, MCP, etc.

Follow-up conversation turned that into persistent notes in this lab (`architecture`, `layered-design`) — bilingual, with ASCII diagrams and links into the real repo.

## What Worked

- **Low barrier to start** — one prompt replaces hours of blind directory browsing
- **Five questions = reusable checklist** for any repo, not just codex
- **Map before microscope** — architecture and flow first; file-level reading comes after you know where to look
- **Notes as cache** — capture the overview in `notes/` so the next session does not re-pay the exploration cost
- **Agent reads what you would skip** — `BUILD.bazel`, workspace `Cargo.toml`, protocol crates — and synthesizes

## What Needed Guidance

- Verifying diagrams against the actual code (agent can miss renames or new crates)
- Choosing how deep to go in follow-ups — one layer per turn beats "explain everything"
- Deciding what belongs in lab `notes/` vs upstream docs (this repo is third-party; notes live here)

## Takeaways

```text
Phase A:  learn-repo-overview prompt  →  mental model (what / arch / folders / flow / stack)
Phase B:  follow-up zoom              →  one path, one layer, one change scenario
Phase C:  notes/                      →  bilingual cache for future you + Codex
```

**Pattern:** learning source used to mean heroic manual reading. With an agent in the workspace, start with a **structured overview prompt** — then read code *with a map in hand*.

Reusable template: [learn-repo-overview.md](../../prompts/learn-repo-overview.md).