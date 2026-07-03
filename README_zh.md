# Codex Labs

[English](README.md) | **中文**

我和 Codex 共用的学习笔记仓库。

这里用来理解 Codex 如何工作、记录实用工作流，并收集在软件项目、写作、研究、自动化和日常问题解决中使用 Codex 的真实案例。

## 目的

这个仓库帮助我：

- 在真实项目里使用 Codex，从而学习它。
- 记录有用的 Codex 工作流和模式。
- 保存 prompt、任务、结果和经验教训。
- 建立一份个人参考：什么场景下 Codex 最好用。
- 探索人类判断和代理式编程如何协作。

## 这里放什么

仓库可以自然生长。有用的内容包括：

- Codex 概念、功能和心理模型的笔记。
- 真实 Codex 会话的案例研究。
- Prompt 示例和可复用的任务描述。
- 小实验、原型和 demo。
- 反思：Codex 做得好的地方，以及哪里需要人工引导。
- 常见工作流清单：调试、review、写文档、交付改动等。

## 建议结构

```text
codex-labs/
  README.md
  notes/
    architecture.md
    architecture_cn.md
    layeredDesign.md
    layeredDesign_cn.md
  cases/
    001-first-session.md
  prompts/
    review-prompts.md
    debugging-prompts.md
  experiments/
```

结构可以随使用方式演变。重点是每份文档都让未来的 Codex 工作更容易理解、复用和改进。

## 文档

| 主题 | English | 中文 |
|------|---------|------|
| 架构总览 | [architecture.md](notes/architecture.md) | [architecture_cn.md](notes/architecture_cn.md) |
| 分层架构详解 | [layeredDesign.md](notes/layeredDesign.md) | [layeredDesign_cn.md](notes/layeredDesign_cn.md) |

## 案例模板

记录一次 Codex 会话时，可以用这个轻量格式：

```markdown
# Case: <short title>

## Context

项目、问题或目标是什么？

## What I Asked Codex

我给了什么 prompt 或指令？

## What Codex Did

涉及哪些文件、命令、推理或输出？

## What Worked

哪些有用、准确或出乎意料地有效？

## What Needed Guidance

哪些地方需要我澄清、纠正、review 或做最终决定？

## Takeaways

下次应该记住什么？
```

## 工作原则

- 把 Codex 当作协作者，而不是自动驾驶。
- 写下让会话成功的上下文。
- 优先具体例子，而不是抽象建议。
- 笔记保持短小，方便复用。
- 失败也要记录；它们往往最能教会你工作流。

## 状态

仓库刚刚起步。随着真实会话、示例和反思的积累，它会越来越有用。