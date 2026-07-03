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

| Location | Required files |
| -------- | -------------- |
| `skills/<name>/` | `README.md`, `README_zh.md`, `SKILL.md`, `checklist.md` |
| `workflows/<name>/` | `README.md`, `README_zh.md`, `WORKFLOW.md` |
| `prompts/` | `<skill-name>.md` triggering that skill |

## Principles

Prioritize: small reusable docs, low decision cost, concrete cases over abstract advice.

Avoid: duplicating openai/codex docs here, huge single markdown files, skills that are secretly full workflows.