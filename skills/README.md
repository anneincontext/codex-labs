# Skills

**English** | [中文](README_zh.md)

Reusable **capabilities** — atomic things Codex (and you) can do well.

```text
skills/
├── debugging/           # narrow repro, logs, root cause, fix proposals
├── code-review/         # read diffs, risks, style, merge readiness
├── repo-understanding/  # map layout, entry points, conventions
├── documentation/       # READMEs, guides, clear prose
├── architecture/        # system design notes, diagrams, tradeoffs
└── testing/             # test plans, coverage gaps, flaky tests
```

| Skill | Folder | Status |
| ----- | ------ | ------ |
| Debugging | [debugging/](debugging/) | SKILL + checklist + [prompt](../prompts/debugging.md) |
| Code review | [code-review/](code-review/) | README only |
| Repo understanding | [repo-understanding/](repo-understanding/) | README only |
| Documentation | [documentation/](documentation/) | README only |
| Architecture | [architecture/](architecture/) | README only |
| Testing | [testing/](testing/) | README only |

Copy [_template/](_template/) when adding a skill: `SKILL.md` + `checklist.md`.

## How skills relate to the lab

```text
prompts/     ──trigger──►  skills/
skills/      ──combine──►  workflows/
cases/       ──record───►   real runs
notes/       ──explain──►   theory behind skills
```