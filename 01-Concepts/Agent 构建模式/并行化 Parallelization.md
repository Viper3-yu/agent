---
title: 并行化 Parallelization
aliases: [Parallelization, 并行执行]
tags:
  - concept
  - topic/agent
  - status/seed
category: workflow
complexity: medium
source: "Anthropic: Building Effective Agents"
created: 2026-06-12
updated: 2026-06-12
---

# 并行化 Parallelization

> 同时执行多个独立子任务，最后汇总结果。两种变体：分段并行（Sectioning）和投票并行（Voting）。

## 模式定义

Parallelization 模式将任务拆分为多个可同时执行的子任务，或对同一任务进行多次独立执行，然后通过汇总或投票机制合并结果。它包含两种变体：Sectioning（分段并行）将大任务拆为独立子任务并行处理；Voting（投票并行）对同一任务多次执行取共识，提升可靠性。

## 适用场景

1. **Sectioning**：多维度分析（情感 + 主题 + 关键词同时提取）
2. **Sectioning**：多语言翻译（各语种并行翻译）
3. **Sectioning**：并行搜索多个数据源后汇总
4. **Voting**：内容审核（多次判断取多数一致）
5. **Voting**：安全检查（多维度并行验证）
6. **Voting**：需要高可靠性的关键决策

## 不适用场景

1. 子任务之间有强顺序依赖 — 应使用 [[Prompt Chaining]]
2. 子任务执行时间差异极大 — 短任务等待长任务，浪费资源
3. 需要动态决定执行哪些子任务 — 应使用 [[编排者-工人模式 Orchestrator-Worker]]

## 架构图

### Sectioning（分段并行）

```
输入 → ┬─ [子任务A] ─→ 结果A ─┐
       ├─ [子任务B] ─→ 结果B ─┼→ [汇总] → 最终输出
       └─ [子任务C] ─→ 结果C ─┘
```

### Voting（投票并行）

```
输入 → ┬─ [执行1] ─→ 结果1 ─┐
       ├─ [执行2] ─→ 结果2 ─┼→ [投票/聚合] → 最终输出
       └─ [执行3] ─→ 结果3 ─┘
```

## 核心流程

1. 判断任务是否可拆分为独立子任务（Sectioning）或需要多次执行（Voting）
2. 将子任务分发到并行执行通道
3. 等待所有通道完成（或设置超时）
4. **Sectioning**：汇总各子任务结果，合并为最终输出
5. **Voting**：对多次执行结果取多数一致或加权聚合

## 伪代码示例

```python
import asyncio

# Sectioning: 分段并行
async def sectioning(input_text):
    tasks = [
        extract_sentiment(input_text),
        extract_topics(input_text),
        extract_keywords(input_text),
    ]
    results = await asyncio.gather(*tasks)
    return merge_results(results)

# Voting: 投票并行
async def voting(input_text, num_votes=3):
    tasks = [
        call_llm(input_text) for _ in range(num_votes)
    ]
    results = await asyncio.gather(*tasks)
    return majority_vote(results)
```

## 优势

- 显著降低总延迟（并行执行 vs 顺序执行）
- Voting 通过多次执行提升结果可靠性
- Sectioning 充分利用独立子任务的并行性
- 各通道互不干扰，单通道失败不影响其他通道

## 局限与风险

- 并行执行增加 API 调用次数，成本上升
- Voting 需要多次调用，成本成倍增加
- 汇总逻辑本身可能成为瓶颈
- 子任务结果冲突时，合并策略需要仔细设计

## 关键决策点

| 决策 | 建议 | 理由 |
|------|------|------|
| Sectioning vs Voting | 子任务独立选 Sectioning，需高可靠选 Voting | 根据任务特性选择变体 |
| 并行度 | 控制在 3-5 个通道 | 过多通道增加成本且汇总复杂 |
| Voting 次数 | 奇数次（3 或 5） | 避免平票 |
| 超时策略 | 设置单通道超时 | 避免慢通道阻塞整体 |
| 汇总方式 | 结构化合并 or 投票 | Sectioning 用合并，Voting 用投票 |

## 与其他模式的关系

- 与 [[Prompt Chaining]] 可组合：链中某步骤内部并行化
- [[评估者-优化者模式 Evaluator-Optimizer]] 中 Voting 的延伸应用
- 可被 [[编排者-工人模式 Orchestrator-Worker]] 使用：编排者决定并行分配子任务
- [[路由模式 Routing]] 的每条路径可内部并行化

## 参考资料
- [[构建模式总览]]
- [[评估者-优化者模式 Evaluator-Optimizer]]（Voting 的延伸）
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
