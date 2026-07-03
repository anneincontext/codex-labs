# 真实世界里的 AI 工作流（田野调研）

[English](real-workflows.md) | **中文**

来自三次语音访谈的提炼。完整记录：[002-biotech-swe](../cases/002-biotech-swe/case_zh.md) · [003-fintech-doc-pipeline](../cases/003-fintech-doc-pipeline/case_zh.md) · [004-bigtech-infra](../cases/004-bigtech-infra/case_zh.md)。

## 跨案例模式

### 1. 每一跳 AI 转换都有人掌舵

三个案例都不是「自动驾驶」。每个转换步骤都有**角色负责人**：

| 步骤 | 典型负责人 |
| ---- | ---------- |
| 需求 → 工单 / 文档 | 产品经理 |
| 文档 → 技术方案 | 开发 |
| 方案 → 代码 | 开发 |
| 代码 → 验证 / 交付 | 人跑 CI/CD、测试、MR |

**Lab 对应：** `skills/*/checklist.md` + `workflows/*/WORKFLOW.md` 里的人工关卡。

### 2. 采纳往往是认知问题，不只是工具问题

生物科技案例：同事仍手动解 Git 冲突，因为**不知道** agent 可以这么做。

**对 codex-labs 的启示：** 要写清 agent **能做什么**（skills 索引），不只写 prompt 技巧。

### 3. 文档即契约（金融科技）

需求文档 A（写代码前）vs 需求文档 B（从代码反推）→ **对比**，PM + 开发共同 steer。AI 搬文字；人对齐负责。

**Lab 对应：** 未来可加 workflow `doc-driven-delivery/`；skill `documentation/`。

### 4. Harness 强领域相关（大厂 infra）

不能脱离 **领域** 谈 harness / agent loop：测试平台、可观测性、中间件状态、运维手册。

**Lab 对应：** [agents-md_cn.md](agents-md_cn.md) 是项目级；组织级 harness = 分领域的 skills + 评测。

### 5. 生产力 ↑ ≠ 工作量 ↓

金融科技：编码 agent 提高了产出；**总负担没明显变轻**（合规、评审、文档链）。

### 6. 成熟度阶梯（infra 工程师）

不是纯升级路径——**手写代码现在依然存在**，与后面几层并行：

```text
手写代码（日常仍常见）
    → Human-in-the-loop（harness 不够；环境给不了下一步信号）
        → Multi-agent（工程师定目标、设计流水线）
```

**Lab 对应：** `workflows/` = 设计好的流水线；`AGENTS.md` = 循环约束。

## 野外看到的约束

| 约束 | 案例 |
| ---- | ---- |
| 机密 / 基因数据 | 生物科技 |
| 金融合规 | 金融科技 |
| 企业工具额度不够（Cursor $120/人仍要自己订阅） | 生物科技 |
| 模型访问（Claude 被封、共享中转账号） | 大厂 / 国内 |

## 提到的工具

- Cursor enterprise + 个人订阅叠加
- Codex + 钉钉文档
- 全局 `AGENTS.md`
- codex-cli、OpenClaw

## 案例

- [002-biotech-swe](../cases/002-biotech-swe/case_zh.md) · [003-fintech-doc-pipeline](../cases/003-fintech-doc-pipeline/case_zh.md) · [004-bigtech-infra](../cases/004-bigtech-infra/case_zh.md)