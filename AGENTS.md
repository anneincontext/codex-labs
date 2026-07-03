# Codex Labs — working agreements

Rules for Codex (and humans) maintaining this repository.

## Always

- Keep **skills** atomic; put multi-step flows in **workflows** only.
- Add bilingual pairs: `README.md` + `README_zh.md` for every new folder.
- Link outward to [openai/codex](https://github.com/openai/codex) with full GitHub URLs in `resources/`.
- Record a real run in `cases/` when a workflow proves useful (or fails instructively).

## Before adding content

- Prefer extending an existing skill/workflow folder over creating a near-duplicate.
- New skill → must map to a `prompts/<name>.md` (even if stub).
- New external project reference → `resources/links.md` + `links_zh.md`.

## File conventions

| Location              | Required files                                              |
| --------------------- | ----------------------------------------------------------- |
| `skills/<name>/`      | `README.md`, `README_zh.md`, `SKILL.md`, `checklist.md`     |
| `workflows/<name>/`   | `README.md`, `README_zh.md`, `WORKFLOW.md`                  |
| `prompts/`            | `<skill-name>.md` triggering that skill                     |

## Tables and diagrams

**Review alignment before you finish** — messy tables are a common rework source in this repo.

### Markdown tables

- Every column gets a **named header** (no empty header cells like `| Case | |`).
- Separator row dashes **match header column width**.
- Pad cell text so the closing `|` lines up in a **monospace** view.
- **Chinese / mixed text:** pad by **display width** (CJK characters count as 2; ASCII as 1), not raw character count.
- Long descriptions belong in the second column; keep a compact `· [中文]` / `· [English]` lang link at the end.

### Flow diagrams — use [D2](https://d2lang.com/)

- Do **not** hand-draw ASCII box diagrams for multi-step flows — they are hard to align (especially with Chinese text).
- Put sources in `notes/diagrams/<topic>/` as `.d2` files; commit matching `.svg` for GitHub preview.
- Render: `d2 --layout dagre file.d2 file.svg` (see [diagrams/learn-codebase-flow/README.md](notes/diagrams/learn-codebase-flow/README.md)).
- Bilingual notes: English `foo.d2` + `foo.svg` and Chinese `foo_cn.d2` + `foo_cn.svg` when labels differ.
- Embed in markdown: `![caption](diagrams/.../foo.svg)` plus a **Source:** link to the `.d2` file.
- Example: [notes/learn-codebase-flow.md](notes/learn-codebase-flow.md).

### Before committing

- Check table right edges in a fixed-width editor.
- After editing `.d2`, re-render and commit the `.svg` too.

## Principles

Prioritize: small reusable docs, low decision cost, concrete cases over abstract advice.

Avoid: duplicating openai/codex docs here, huge single markdown files, skills that are secretly full workflows.