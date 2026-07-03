# 案例：AI-native 仓库的 Knowledge Engineering

[English](case.md) | **中文**

**类型：** Markdown 经验总结 · 笔记整理  
**来源：** Anne 在小红书看到的 OpenAI 团队 Codex 使用技巧分享的 Markdown 总结 + 她自己的理解  
**原始链接：** https://www.xiaohongshu.com/explore/6a3929fb0000000020038fdb?source=webshare&xhsshare=pc_web&xsec_token=CB0zyTNhZ5-qf6Zz-GvOJje0Jyh18zo28MgAfu1Lme0cI=&xsec_source=pc_share

## Prompt / 产出的 Notes

- **输入笔记：** Anne 的完整 Markdown 总结——分享中的两个核心观点，加上她自己的四层模型和 AI-first 仓库草图。
- **相关笔记：** [method/agents-md_zh.md](../../notes/method/agents-md_zh.md) _（AGENTS.md 约束什么——本案例说的是它不该变成什么）_ · [codex/architecture_zh.md](../../notes/codex/architecture_zh.md)

## 背景

这篇分享的元观点只有一句：

> AI 项目的关键已经不是 **Prompt Engineering**，而是 **Knowledge Engineering**。

好的 AI 项目不是 prompt 写得漂亮，而是**整个仓库对 agent 足够友好**。支撑它的是两条具体原则：

### 1. AI 控制执行，人控制知识

| AI 负责（执行层）             | 人负责（知识层）         |
| ----------------------------- | ------------------------ |
| shell 命令、改代码            | `AGENTS.md`、`docs/`     |
| 搜索网页、调用工具            | skills、subagents        |
| 沙箱执行、context compression | 仓库结构、编码约定、架构 |

右边这一列决定了 AI 如何理解整个项目——而且只有人能持续维护。

### 2. Progressive Disclosure，而不是三千行的 AGENTS.md

反模式：把项目介绍 + 架构 + 数据库 + API + 前后端 + 测试 + 部署 + 风格全塞进一个 `AGENTS.md`，最后膨胀到两三千行。

更好的形态——`AGENTS.md` 是**目录**（约 100–300 行），路由到聚焦的文档：

```text
AGENTS.md                    docs/
├── Project Overview         ├── architecture.md
├── Repository Structure     ├── backend.md · frontend.md
└── Where to Find Things ──► ├── testing.md · deployment.md
                             └── api.md
```

于是每个任务只加载真正需要的知识：

```text
「实现登录」→ 读 AGENTS.md → Authentication → docs/authentication.md → 实现
```

而不是 `Load Everything`。效果：context 更小、成本更低、更准确、更容易维护。

## 原分享之外的解读

| 层                     | 给谁 | 内容                                                                           | 回答什么问题         |
| ---------------------- | ---- | ------------------------------------------------------------------------------ | -------------------- |
| 1 · 项目结构           | 人   | `src/ tests/ scripts/ docs/`                                                   | 仓库好维护吗？       |
| 2 · 知识层             | AI   | `AGENTS.md`（只导航）+ `docs/` + skills + subagents                            | **AI 知道什么？**    |
| 3 · Workflow / Harness | AI   | `/implement` `/review` `/refactor` `/debug`——各自选 skill、subagent、MCP、模型 | **AI 怎么工作？**    |
| 4 · 反馈闭环           | 系统 | 生成 → 验证 → 评估 → 改进 workflow → 更新 skill                                | 系统会随时间变好吗？ |

第 4 层是未来最重要的一层：workflow 不断优化、skill 不断更新、仓库知识越来越完整——一个真正的闭环。

评论区两条补充：

- _「Harness Engineering 太重要了，是真正把玩具项目变成工程项目的方法。」_
- 一个实践：**每次 Codex 执行前后自动扫描项目目录、更新结构文档**——知识自动维护，而不是手动。

## AI-first 仓库的目标形态

```text
repo/
├── AGENTS.md        # 只负责导航（100–300 行）
├── docs/            # 长期知识
├── skills/          # 可复用能力
├── workflows/       # 任务流程（feature / bugfix / review / release）
├── templates/       # issue / pr / adr
├── evals/           # 验证 AI 是否真的做好
├── scripts/         # 自动维护仓库知识（如 update_project_map）
├── src/  └── tests/
```

仓库不再只是代码仓库——它同时是 AI 的**知识库、操作手册和工作流系统**。

## 收获

```text
旧模型：
  更好的 prompt → 更好的回答

AI-first 仓库模型：
  更好的仓库知识
      → 更好的 agent 上下文
      → 更好的执行
      → 更好的反馈
      → 更好的仓库知识（闭环）
```

**模式：** 把 `AGENTS.md` 做成路由器，不是仓库。长期知识放聚焦文档；可复用行为放 skills/workflows；质量检查放 evals；维护交给脚本自动化。agent 按当前任务渐进披露所需上下文。

未来优秀的工程团队，比拼的不是模型能力，而是谁能构建更好的 **AI-Native Engineering System**。
