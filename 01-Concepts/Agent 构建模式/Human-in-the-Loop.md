---
title: Human-in-the-Loop
aliases: [HITL, 人机协作, 人在回路]
tags:
  - concept
  - topic/agent
  - status/seed
category: safety
complexity: medium
source: "Anthropic: Building Effective Agents"
created: 2026-06-12
updated: 2026-06-12
---

# Human-in-the-Loop

> 在 Agent 自动化流程的关键节点引入人工审批、确认或干预。平衡自动化效率与风险控制。

## 模式定义

Human-in-the-Loop（HITL）模式在 Agent 工作流的关键决策点插入人工介入环节，通过审批、确认或纠错三种方式确保高风险操作的安全性。它不是独立的处理模式，而是叠加在其他模式之上的安全层，用于在自动化效率和风险控制之间取得平衡。

## 适用场景

1. 高风险写操作（修改数据库、删除文件、发送邮件）
2. 外部通信（发送消息给客户、发布内容）
3. 资金相关操作（大额交易、支付审批）
4. 合规要求必须有人工审批的流程
5. Agent 遇到无法处理的异常情况需要人工判断

## 不适用场景

1. 低风险只读操作（查询、分析、内部计算）— 直接自动执行
2. 对延迟极度敏感的实时系统 — 人工介入引入不可控延迟
3. 操作频率极高且单次风险极低 — 人工审批成为瓶颈
4. 已有充分的自动化安全检查可以替代 — 不需要额外人工层

## 架构图

```
Agent 工作流
    ↓
[决策节点: 是否需要人工介入?]
    │
    ├─ 低风险 → 自动执行 → 继续工作流
    │
    └─ 高风险 → [暂停执行]
                    ↓
              [通知人工: 提供上下文 + 推荐方案]
                    ↓
              [等待人工决策]
                    ├─ 批准 → 执行 → 继续工作流
                    ├─ 拒绝 → 取消/回滚
                    └─ 修改 → 按人工指示调整后执行
```

## 核心流程

1. Agent 工作流到达需要判断的决策节点
2. 评估操作风险等级（高/中/低）
3. 低风险操作自动执行，高风险操作暂停
4. 向人工提供充分上下文（操作内容、影响范围、Agent 推荐方案）
5. 等待人工决策（批准/拒绝/修改），设置超时机制
6. 根据人工决策继续执行或回滚

## 伪代码示例

```python
def human_in_the_loop(operation, context):
    risk_level = assess_risk(operation)

    if risk_level == "low":
        return execute(operation)

    # 高风险：暂停并请求人工介入
    notification = {
        "operation": operation.description,
        "impact": operation.impact_analysis,
        "recommendation": agent_recommendation(operation),
        "context": context,
    }
    human_decision = notify_and_wait(notification, timeout=300)

    if human_decision.action == "approve":
        return execute(operation)
    elif human_decision.action == "reject":
        return rollback(operation)
    elif human_decision.action == "modify":
        adjusted = apply_human_modifications(operation, human_decision.changes)
        return execute(adjusted)
```

## 三种介入模式

| 模式 | 说明 | 场景 |
|------|------|------|
| **审批** | Agent 提出方案，人批准后执行 | 发送邮件、修改数据库 |
| **确认** | Agent 请求确认关键决策 | 删除文件、大额交易 |
| **纠错** | Agent 遇到无法处理的情况求助 | 异常输入、安全疑虑 |

## 优势

- 高风险操作有安全兜底，防止不可逆错误
- 人工判断可以处理 Agent 无法覆盖的边界情况
- 提供审计追踪，满足合规要求
- 逐步建立对 Agent 的信任，从审批到自动化的渐进过程

## 局限与风险

- 人工介入引入不可控延迟（响应时间依赖人工）
- 过多介入点会大幅降低自动化效率
- 人工可能因疲劳或信息不足做出错误决策
- 需要设计良好的通知和上下文展示机制

## 关键决策点

| 决策 | 建议 | 理由 |
|------|------|------|
| 介入点选择 | 仅在高风险操作前插入 | 平衡安全性和效率 |
| 风险分级标准 | 写操作 > 读操作，外部 > 内部，资金 > 非资金 | 明确哪些需要人工审批 |
| 超时策略 | 设置 5-10 分钟超时，超时后默认拒绝 | 避免阻塞，安全优先 |
| 上下文展示 | 提供操作详情 + 影响分析 + 推荐方案 | 帮助人工快速准确决策 |
| 渐进自动化 | 先审批后观察，逐步减少介入频率 | 建立信任，提升效率 |

## 设计原则

1. **高风险操作必须人工审批**（写操作、外部通信、资金相关）
2. **低风险操作自动执行**（读取、分析、内部计算）
3. **提供充分上下文**给人工决策
4. **设置超时机制**，避免阻塞

## 在框架中的支持

- Microsoft Agent Framework：原生 HITL
- LangGraph：Checkpoint + Interrupt 机制
- Claude Code：工具调用确认

## 与其他模式的关系

- 可叠加在任何模式之上，作为安全层
- [[编排者-工人模式 Orchestrator-Worker]] 可在编排者决策节点引入人工审批
- [[评估者-优化者模式 Evaluator-Optimizer]] 可在评估者环节引入人工判断
- [[路由模式 Routing]] 可在路由结果不确定时请求人工确认
- [[Prompt Chaining]] 可在 gate check 失败时请求人工介入

## 参考资料
- [[构建模式总览]]
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
