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

### Flow diagrams — use [Mermaid](https://mermaid.js.org/)

- Do **not** hand-draw ASCII box diagrams for multi-step flows — they are hard to align (especially with Chinese text).
- Embed inline in markdown: a fenced ` ```mermaid ` code block (GitHub renders it natively).
- Bilingual notes: separate ` ```mermaid ` blocks per language when labels differ (EN doc vs CN doc).
- Keep diagrams focused — one flow per block; prefer `flowchart TD` for step-by-step paths.
- Example: [notes/method/learn-codebase-flow.md](notes/method/learn-codebase-flow.md).

### Before committing

- Check table right edges in a fixed-width editor.
- Preview Mermaid on GitHub or in an editor with Mermaid support before pushing.

## Principles

Prioritize: small reusable docs, low decision cost, concrete cases over abstract advice.

Avoid: duplicating openai/codex docs here, huge single markdown files, skills that are secretly full workflows.