# Prompt: Implement Demo (Realtime AI Presenter)

**English** | [中文](implement-demo_zh.md)

**Skill:** [implement-feature](../workflows/implement-feature/) + [repo-understanding](../skills/repo-understanding/)  
**Case:** [cases/001-realtime-ai-presenter/case.md](../cases/001-realtime-ai-presenter/case.md)  
**Resulting repo:** [realtime-ai-presenter](https://github.com/anneincontext/realtime-ai-presenter)

Bootstrap prompt Anne used with Codex to build the demo.

---

## Template (full)

```text
Objective:
Build a public-launch-style demo based on openai-realtime-console: an interruptible
voice-driven AI presenter for slides.

Base repo:
Use https://github.com/openai/openai-realtime-console as the foundation. Preserve the
Realtime API WebRTC connection, audio flow, data channel events, and logging panel
where useful.

Demo concept:
The user starts a presentation. The AI presenter explains the current slide using voice.
The user can interrupt naturally and say things like:
- "next slide"
- "go back"
- "explain this in simpler terms"
- "skip to the demo architecture slide"

Tech stack preferences:
- Keep React + TypeScript
- Keep Vite if the repo already uses it
- Prefer pnpm
- Avoid adding heavy dependencies
- Use simple local slide data first
- Keep architecture readable for a Developer audience

Success criteria:
- App runs locally
- User can connect to Realtime session
- AI speaks as a presenter
- Voice interruption works
- Slide navigation works via function calling
- UI looks like a small product demo, not just a console
- Event/debug panel remains available to show Realtime API behavior

Constraints:
- Do not rewrite the entire app
- Do not over-engineer slide upload
- Hardcode 5–7 demo slides if needed
- Prioritize reliability over completeness
- Label shortcuts clearly in README

Verification:
- pnpm install
- pnpm dev
- Manual test: connect, start presenting, interrupt, navigate slides
```

## What this prompt constrains (vs AGENTS.md)

| Section | Role |
| ------- | ---- |
| Objective / Demo concept | **What to build** — product shape |
| Base repo | **Where to start** — fork/extend, not greenfield |
| Tech stack preferences | **Stack choices** for this session |
| Success criteria | **Definition of done** — testable outcomes |
| Constraints | **Scope limits** — anti-rewrite, anti-over-engineer |
| Verification | **How to prove it works** |

After the repo exists, [AGENTS.md](https://github.com/anneincontext/realtime-ai-presenter/blob/main/AGENTS.md) takes over for **every later session** (tests, pnpm, no surprise deps, design principles).

## Observed rhythm (Anne's Realtime demo)

- **~20 minutes:** this prompt once → first demoable version
- **After:** continued conversation for usability polish (small prompts, small diffs) — no need to repeat the full spec

Fits the bootstrap step of [implement-feature](../workflows/implement-feature/); polish uses short turns + `AGENTS.md`.