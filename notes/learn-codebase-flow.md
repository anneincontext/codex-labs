# Learn a codebase — prompt to result

**English** | [中文](learn-codebase-flow_cn.md)

How a **learning session** chains together: your prompt, what Codex does at runtime, and the durable output in `notes/`.

> Case: [005-learn-codex-repo](../cases/005-learn-codex-repo/case.md) · Prompt: [learn-repo-overview.md](../prompts/learn-repo-overview.md) · Example output: [architecture.md](architecture.md)

---

## Two chains (stacked)

| Chain             | Question it answers                                    |
| ----------------- | ------------------------------------------------------ |
| **Lab workflow**  | What *you* do across turns — overview → zoom → notes   |
| **Codex runtime** | What happens *inside one turn* — input → tools → reply |

You care about both: the runtime produces the chat answer; the lab workflow turns it into something you keep.

---

## 1. Lab workflow (prompt → durable result)

```mermaid
flowchart TD
  subgraph phaseA["Phase A — Bootstrap"]
    A1["Open target repo in Codex workspace"]
    A2["learn-repo-overview prompt\n(what / arch / folders / flow / stack)"]
    A3["Mental model in chat"]
    A1 --> A2 --> A3
  end

  subgraph phaseB["Phase B — Zoom"]
    B1["Diagram aligned to folder names"]
    B2["One user turn end-to-end"]
    B3["If changing X — read what first?"]
    B4["One layer or one path per turn"]
    B1 --> B4
    B2 --> B4
    B3 --> B4
  end

  subgraph phaseC["Phase C — Cache"]
    C1["Write or refine notes/\n(e.g. architecture, layeredDesign)"]
    C2["Reuse next session — no re-exploration cost"]
    C1 --> C2
  end

  phaseA --> phaseB --> phaseC
```

| Phase | Input                       | Output                            |
| ----- | --------------------------- | --------------------------------- |
| A     | One structured prompt       | Chat overview (five sections)     |
| B     | Small follow-up prompts     | Deeper slices (diagram, one path) |
| C     | Your edit / “save to notes” | Files under `notes/`              |

The **first result** is the agent reply. The **result worth keeping** is Phase C.

---

## 2. Codex runtime (one learning turn)

When you ask for a repo overview, each turn roughly follows this path (see [architecture.md](architecture.md) for the full system):

```mermaid
flowchart TD
  you["You\n(prompt)"] --> cli["CLI / TUI"]
  cli --> core["codex-core\nOp::UserInput"]
  core --> context["Build context\n(AGENTS.md, cwd, thread, skills…)"]
  context --> api["OpenAI Responses API\n(streaming)"]
  api --> tools["Read / list / search repo"]
  tools --> back["Tool results → model"]
  back --> api
  api --> ui["Stream events to UI"]
  ui --> done["Turn complete\nthread persisted"]
```

**One line:**

```text
prompt → UserInput → context → model ↔ tools(repo) → streamed reply → (you) notes/
```

For **codebase learning**, the tool loop matters: the model reads `README`, manifests, crate layout, entry points — then synthesizes your five bullets. It is not only pretraining.

### IDE / SDK variant

Same core loop over JSON-RPC (`app-server`): `thread/start` → `turn/start` → event stream → `turn/completed`. See [architecture.md § How Requests Flow](architecture.md#how-requests-flow-through-the-system).

---

## 3. How the chains connect

```mermaid
flowchart TD
  prompt["Overview prompt\n(5 questions)"] --> turn["Codex turn(s)\nUserInput → context → API ↔ tools(repo)"]
  turn -->|immediate| chat["Chat answer\n(immediate)"]
  turn -->|your follow-ups| notes["notes/\n(durable)"]
```

[Case 004](../cases/004-bigtech-infra/case.md): interviewee said Codex helps with **code interpretation**. [Case 005](../cases/005-learn-codex-repo/case.md): Anne tried it on [openai/codex](https://github.com/openai/codex) → **codex-labs** caches the notes.

---

## 4. Minimal recipe

```text
1. Open repo in Codex
2. One overview prompt (map, not line-by-line)
3. Verify + zoom (diagram, one request path, “where to change X”)
4. Write notes/ — that is the result you keep
5. Next session: read notes first, then ask targeted questions
```

---

## Related

| Doc                                                         | Role                                  |
| ----------------------------------------------------------- | ------------------------------------- |
| [learn-repo-overview.md](../prompts/learn-repo-overview.md) | Copy-paste bootstrap prompt           |
| [architecture.md](architecture.md)                          | What Codex *is* (learned via this flow) |
| [layeredDesign.md](layeredDesign.md)                        | Phase B zoom on Codex layers          |
| [agents-md.md](agents-md.md)                                | Ongoing constraints after bootstrap   |