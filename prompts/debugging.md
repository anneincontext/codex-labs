# Prompt: Debugging

**Skill:** [skills/debugging/](../skills/debugging/)

## Template

```text
Debug this issue in {{project}}.

Symptom: {{symptom}}
Repro (if known): {{repro}}

1. Confirm repro with the project's test/lint commands (read AGENTS.md if present).
2. Identify root cause with file:line evidence.
3. Propose the smallest fix.
4. Add or update a regression test.

Do not add dependencies without asking.
```

## Parameters

- `{{project}}` — repo name or path
- `{{symptom}}` — what breaks
- `{{repro}}` — steps or command