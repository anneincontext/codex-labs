# Prompt：实现 Demo（Realtime AI Presenter）

[English](implement-demo.md) | **中文**

**Workflow：** [implement-feature](../workflows/implement-feature/) + [repo-understanding](../skills/repo-understanding/)  
**案例：** [cases/001-realtime-ai-presenter/case_zh.md](../cases/001-realtime-ai-presenter/case_zh.md)  
**产出仓库：** [realtime-ai-presenter](https://github.com/anneincontext/realtime-ai-presenter)

Anne 用 Codex 启动该 demo 时的 bootstrap prompt。

---

## 模板（全文）

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

## 这份 prompt 约束了什么（对比 AGENTS.md）

| 段落 | 作用 |
| ---- | ---- |
| Objective / Demo concept | **做什么** — 产品形态 |
| Base repo | **从哪起步** — 扩展而非从零写 |
| Tech stack preferences | **本次会话的技术选型** |
| Success criteria | **完成标准** — 可验证结果 |
| Constraints | **范围边界** — 禁止重写、禁止过度设计 |
| Verification | **如何验收** |

仓库建好后，[AGENTS.md](https://github.com/anneincontext/realtime-ai-presenter/blob/main/AGENTS.md) 负责**之后每次会话**（测试、pnpm、加依赖先问、设计原则）。

## 实测节奏（Anne 的 Realtime demo）

- **~20 分钟**：本 prompt 一次 → 可演示的第一版
- **之后**：持续对话打磨可用性（小 prompt、小改动），不必重写任务书

适合 [implement-feature](../workflows/implement-feature/) workflow 的「启动」阶段，打磨阶段用短指令 + `AGENTS.md` 即可。