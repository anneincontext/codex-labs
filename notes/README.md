# Notes

**English** | [中文](README_zh.md)

Theory, mental models, and research notes — **why** Codex works the way it does.

Not session writeups (`cases/`), not runnable templates (`prompts/`).

## Layout

```text
notes/
├── codex/      # "How is openai/codex built?"  — study of the target repo
├── method/     # "How do I work with agents?"  — transferable methodology
└── research/   # "How do others use AI?"       — field research
```

Every note opens with a header: **Answers** (what question it settles), **Read first** (prerequisite), and — for `codex/` notes only — **Verified against** (the upstream commit the claims were checked on; code notes go stale, method notes don't).

## Start here — pick by goal

| You want to… | Read |
| ------------ | ---- |
| Get the full picture of openai/codex fast | [codex/architecture.md](codex/architecture.md) *(hub — other codex/ notes branch from it)* |
| Understand the crate layering | [codex/layered-design.md](codex/layered-design.md) |
| See how the terminal UI is designed | [codex/tui-interface-design.md](codex/tui-interface-design.md) |
| Look up TUI slash commands / shortcuts | [codex/tui-commands.md](codex/tui-commands.md) |
| Learn a codebase with an agent (the flow this lab uses) | [method/learn-codebase-flow.md](method/learn-codebase-flow.md) |
| Write a good AGENTS.md | [method/agents-md.md](method/agents-md.md) |
| Know how practitioners actually use AI tools | [research/real-workflows.md](research/real-workflows.md) |

## All notes

| Note | English | 中文 |
| ---- | ------- | ---- |
| Architecture overview | [codex/architecture.md](codex/architecture.md) | [codex/architecture_zh.md](codex/architecture_zh.md) |
| Layered design | [codex/layered-design.md](codex/layered-design.md) | [codex/layered-design_zh.md](codex/layered-design_zh.md) |
| TUI interface design | [codex/tui-interface-design.md](codex/tui-interface-design.md) | [codex/tui-interface-design_zh.md](codex/tui-interface-design_zh.md) |
| TUI commands | [codex/tui-commands.md](codex/tui-commands.md) | [codex/tui-commands_zh.md](codex/tui-commands_zh.md) |
| Learn a codebase (prompt → result) | [method/learn-codebase-flow.md](method/learn-codebase-flow.md) | [method/learn-codebase-flow_zh.md](method/learn-codebase-flow_zh.md) |
| What AGENTS.md constrains | [method/agents-md.md](method/agents-md.md) | [method/agents-md_zh.md](method/agents-md_zh.md) |
| Real-world workflows (3 interviews) | [research/real-workflows.md](research/real-workflows.md) | [research/real-workflows_zh.md](research/real-workflows_zh.md) |

## Conventions

- **English is canonical; `_zh.md` mirrors follow.** Update both in the same change.
- **Hub-and-spoke:** deep dives link back to their hub (`codex/architecture.md`), not to every sibling.
- **`codex/` notes carry a "Verified against" commit** — before trusting a detail, run `git diff <commit>..HEAD -- codex-rs/` upstream to see if it drifted.
