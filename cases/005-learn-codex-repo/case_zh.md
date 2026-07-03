# 案例：用 AI 解读并学习 openai/codex

[English](case.md) | **中文**

**类型：** 自己的学习会话（Anne + Codex）  
**来源：** Anne 用 Codex 阅读、学习 [openai/codex](https://github.com/openai/codex) 仓库  
**目标仓库：** [openai/codex](https://github.com/openai/codex)（约 100 个 Rust crate，CLI + TUI + app-server）

## Prompt / 产出的 Notes

- **第一个 prompt：** [prompts/learn-repo-overview_zh.md](../../prompts/learn-repo-overview_zh.md)
- **架构总览：** [notes/architecture_cn.md](../../notes/architecture_cn.md) / [English](../../notes/architecture.md)
- **分层细读：** [notes/layeredDesign_cn.md](../../notes/layeredDesign_cn.md) / [English](../../notes/layeredDesign.md)

## 背景

Codex 是一个体量大、初次接触很陌生的代码库。过去学源码 — 在 crate、`BUILD.bazel`、协议类型之间跳来跳去 — 起步成本太高。

换做法：在 Codex 里打开仓库，先要一份**结构化总览**。让 agent 当**导游**，再决定往哪深读。

## 我向 Codex 提了什么

```text
Give me a high-level overview of this repository.

Please explain:
- what this project does
- the main architecture
- important folders
- how requests flow through the system
- what technologies are used
```

一个 prompt，五个板块。不需要事先知道入口在哪 — 问题本身定义了「第一遍学到什么算完成」。

## Codex 做了什么

- 概括 Codex CLI 做什么（本地 agent、工具、thread、沙箱）
- 画出 `codex-rs/` 的 crate 布局：CLI、TUI、core、app-server、exec、protocol…
- 说明请求流：用户输入 → session → 模型 → 工具 → 沙箱 → 事件回 UI
- 列出技术栈：Rust workspace、Bazel/Cargo、TypeScript 打包、MCP 等

后续对话把这些沉淀成本 lab 里的 `notes/`（`architecture`、`layeredDesign`）— 双语、带 ASCII 图、链到真实仓库路径。

## 哪些有效

- **起步门槛低** — 一个 prompt 换掉几小时盲目翻目录
- **五个问题 = 可复用清单** — 不限于 codex，任何仓库都能用
- **先地图后显微镜** — 先有架构和请求流；知道往哪看再读具体文件
- **Notes 当缓存** — 写进 `notes/`，下次会话不用重新付探索成本
- **Agent 会读你会跳过的文件** — workspace `Cargo.toml`、`BUILD.bazel`、协议 crate — 并帮你综合

## 哪些地方需要我引导

- 对照真实代码核对图表（agent 可能漏掉重命名或新 crate）
- Follow-up 控制深度 — 一层、一条路径、一个回合，比「全讲一遍」靠谱
- 决定什么留在 lab `notes/`、什么该写回上游（第三方仓库；笔记放这里）

## 收获

```text
阶段 A：learn-repo-overview prompt  →  mental model（做什么 / 架构 / 目录 / 流 / 栈）
阶段 B：follow-up  zoom              →  一条路径、一层、一个改动场景
阶段 C：notes/                       →  双语缓存，给未来的你和 Codex
```

**模式：** 过去学源码靠硬啃。工作区里有 agent 时，先用**结构化总览 prompt** — 再**拿着地图**读代码。

可复用模板：[learn-repo-overview_zh.md](../../prompts/learn-repo-overview_zh.md)。