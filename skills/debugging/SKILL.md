# Debugging

## Goal

Find root cause and propose a minimal, test-backed fix.

## Steps

1. **Reproduce** — exact command, cwd, expected vs actual
2. **Collect evidence** — logs, stack trace, last good commit
3. **Hypothesize** — one primary theory with pointers in code
4. **Minimal fix** — smallest diff; avoid drive-by refactors
5. **Verify** — run project test command (see target repo `AGENTS.md`)

## Respect project AGENTS.md

If the repo has `AGENTS.md`, its working agreements override generic habits
(e.g. `npm test` vs `just test`, `pnpm` vs `npm`).