# Prompt：学习代码库 — 高层总览

[English](learn-repo-overview.md) | **中文**

**案例：** [cases/005-learn-codex-repo/case_zh.md](../cases/005-learn-codex-repo/case_zh.md)  
**示例仓库：** [openai/codex](https://github.com/openai/codex)

Anne 用 Codex 学习大型陌生仓库时的第一个 prompt。在 agent 能打开工作区的任何代码库上都适用。

---

## 模板

```text
Give me a high-level overview of this repository.

Please explain:
- what this project does
- the main architecture
- important folders
- how requests flow through the system
- what technologies are used
```

（英文 prompt 对 Codex 通常更稳；需要中文笔记时，下一轮让 agent 用中文复述或整理即可。）

---

## 为什么问这五点

| 问题 | 你会得到什么 |
| ---- | ------------ |
| 项目做什么 | 目的和边界 — 什么在范围内、什么不在 |
| 主架构 | 读具体文件之前的 mental model |
| 重要目录 | 下一步该看哪；避免盲目 `grep` |
| 请求怎么流 | 运行时行为如何把模块串起来 |
| 技术栈 | 语言、构建、依赖 |

这是在要一张**地图**，不是逐行走源码。后续回合可以再 zoom 到某一层、某个 crate 或某条路径。

---

## 建议的 follow-up（总览之后）

```text
画一张 ASCII 请求流图，并和目录名对齐。

跟完一次 user turn 端到端 — 涉及哪些 crate 和文件？

如果要改 [X]，你会先读哪里？
```

把好的回答沉淀到本 lab 的 `notes/`，或你自己仓库的文档里。