# Case: Realtime AI Presenter demo

**English** | [中文](case_zh.md)

**Type:** Own demo (Anne's Codex session)  
**Source:** Anne built this demo herself with Codex

## Skills / Workflow / Prompts

- **Prompt:** [prompts/implement-demo.md](../../prompts/implement-demo.md)
- **Persistent rules:** [AGENTS.md](https://github.com/anneincontext/realtime-ai-presenter/blob/main/AGENTS.md)
- **Repo:** [anneincontext/realtime-ai-presenter](https://github.com/anneincontext/realtime-ai-presenter)

## Context

Build a public-launch-style voice presenter on top of OpenAI's realtime-console — interruptible slides, function calling for navigation, product-ish UI.

## What I Asked Codex

One structured prompt covering: objective, base repo, demo UX, stack prefs, success criteria, constraints, verification. See [implement-demo.md](../../prompts/implement-demo.md).

## Timeline

| Phase | What happened | ~Time |
| ----- | ------------- | ----- |
| **Bootstrap** | One structured prompt ([implement-demo.md](../../prompts/implement-demo.md)) → working demo on top of realtime-console | **~20 min** |
| **Polish** | Continued conversation — usability, edge cases, README shortcuts, presenter flow | Ongoing |

The first pass got something **demoable** quickly because the prompt already specified base repo, success criteria, and verification. Follow-up turns were smaller asks, not re-explaining the whole product.

## What Codex Did

Extended `openai-realtime-console` (WebRTC, audio, data channel) into a slide presenter; added `AGENTS.md` for ongoing sessions.

## What Worked

- **~20 min to first demo** — structured prompt front-loaded decisions (no rewrite debate, clear done bar)
- **Prompt sections map cleanly to decisions** — base repo stopped a rewrite; success criteria = done definition
- **Iterate after bootstrap** — polish via short follow-up prompts beats one giant prompt for every tweak
- **Short AGENTS.md after bootstrap** — cheap to keep in context every turn during polish
- **"Extend, don't rewrite"** in both prompt Constraints and AGENTS.md Principles — aligned message

## What Needed Guidance

- Manual voice/slide testing (Verification section) — especially during polish
- Approving any new production dependencies (AGENTS.md working agreement)
- Judging when polish was "good enough" for a public-launch-style demo

## Takeaways

```text
Phase A (~20 min):  prompt.md     →  demoable skeleton (what / done / verify)
Phase B (ongoing):  conversation  →  usability + feel
Phase C (every edit): AGENTS.md   →  habits / taste / limits
```

**Pattern:** one strong bootstrap prompt, then many small steering turns — not one megaprompt for the whole project.

See [notes/agents-md.md](../../notes/agents-md.md) for the full model.