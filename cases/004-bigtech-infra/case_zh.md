# 案例：大厂 infra — 领域 harness 与成熟度阶梯

[English](case.md) | **中文**

**类型：** 语音访谈 / 田野笔记  
**来源：** 与受访者语音聊天  
**提炼：** [notes/real-workflows_cn.md](../../notes/real-workflows_cn.md)  
**相关案例：** [002-biotech-swe](../002-biotech-swe/case_zh.md) · [003-fintech-doc-pipeline](../003-fintech-doc-pipeline/case_zh.md)

---

## 背景

**大厂 infra 工程师：** 入职不久；架构与业务 **技术债**。消息 / 异步系统。在 infra 里，**工程师就是产品**。约 $1000/月工具 + Cursor Enterprise。

### Harness / 流水线视角

来自小公司的经验：软件生产像 **流水线** — 实践上的 **harness engineering**，但 **强领域相关**：

- 成熟的测试平台
- 可观测的中间件 / 集群状态
- 不能脱离 **业务领域** 谈 harness
- **写代码的 harness** 和 **工作流的 harness** 都重要

### 怎么用 AI

- **运维：** 集群、报警治理、故障响应
- **研发：** 代码生成
- **流水线：** 逐步建设；按领域做 **评测 / benchmark**
- 张力：业务交付压力 vs 体系优化

### 成熟度（他们的说法）

不是纯「升级换代」——**手写代码现在依然存在**，和后面几层并行：

1. 手写代码 — 仍在日常出现
2. Human-in-the-loop — harness 不够好；agent 从环境拿不到下一步信号
3. Multi-agent — **软件工程师定目标、设计流水线**

另有：新人 **onboarding checklist**。

### 个人工具栈

- OpenClaw、**codex-cli**
- 国内 Claude 受限；共享 OpenAI 账号走中转
- Codex：直接聊或 SDK 自管——适合 **大仓库解读**（例如拉 codex 源码来讲）、面试回顾

## 对 codex-labs 的映射

| 观察 | 在 lab 里的位置 |
| ---- | --------------- |
| 领域 harness + 评测 | `skills/` + `experiments/` |
| Anne 20 分钟 demo | [001-realtime-ai-presenter](../001-realtime-ai-presenter/case_zh.md) |