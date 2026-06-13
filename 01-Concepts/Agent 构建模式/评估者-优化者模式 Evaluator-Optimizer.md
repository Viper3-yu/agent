---
title: 评估者-优化者模式 Evaluator-Optimizer
aliases: [Evaluator-Optimizer, 生成-评估循环]
tags:
  - concept
  - topic/agent
  - status/mature
category: workflow
complexity: high
source: "Anthropic: Building Effective Agents"
created: 2026-06-12
updated: 2026-06-12
---

# 评估者-优化者模式 Evaluator-Optimizer

> 一个 Agent 生成结果，另一个 Agent 评估质量，不合格则反馈给生成者重新优化。循环直到满足标准。

## 模式定义

Evaluator-Optimizer 模式通过两个独立 Agent 的协作实现迭代改进：生成者（Generator）产出候选结果，评估者（Evaluator）根据明确的质量标准进行评审并提供反馈，若不合格则将反馈返回给生成者进行优化。循环持续直到结果满足标准或达到最大迭代次数。核心原则是评估者和生成者必须使用不同的 Prompt（甚至不同模型），以避免自我欺骗。

## 适用场景

1. 文案写作（生成 → 评审 → 修改 → 再评审）
2. 代码生成（写代码 → 测试 → 修复 → 再测试）
3. 翻译（翻译 → 质量评估 → 改进）
4. 需要高质量输出且可定义明确评估标准的任务
5. 设计方案迭代（草图 → 评审 → 优化）

## 不适用场景

1. 无法定义明确评估标准的任务 — 评估者无法有效判断
2. 对延迟敏感的场景 — 循环迭代增加多次 LLM 调用
3. 一次生成质量已足够好的简单任务 — 不值得迭代开销
4. 评估成本高于重新生成成本 — 直接重新生成更高效

## 架构图

```
[生成者] → 候选结果 → [评估者] → 评分 + 反馈
    ↑                        │
    │    不合格              │
    └──── 反馈+优化指令 ─────┘
                    │
                    ↓ 合格
                   输出
```

## 核心流程

1. 生成者根据初始 Prompt 或优化反馈产出候选结果
2. 评估者根据明确的质量标准评审候选结果
3. 若合格（评分达标），输出最终结果
4. 若不合格，评估者生成具体反馈和优化建议
5. 将反馈传递给生成者，回到步骤 1
6. 超过最大循环次数时，输出当前最佳结果

## 伪代码示例

```python
def evaluator_optimizer(initial_prompt, eval_criteria, max_iterations=5):
    # 生成者：初始生成
    candidate = call_llm(generator_prompt, initial_prompt)

    for i in range(max_iterations):
        # 评估者：评审候选结果
        evaluation = call_llm(evaluator_prompt, candidate, eval_criteria)

        if evaluation.score >= eval_criteria.threshold:
            return candidate  # 合格，输出

        # 不合格：将反馈传回生成者优化
        candidate = call_llm(
            optimizer_prompt,
            candidate,
            evaluation.feedback
        )

    # 达到最大次数，返回当前最佳
    return candidate
```

## 优势

- 迭代改进显著提升输出质量
- 评估标准明确，改进方向清晰
- 生成者和评估者分离，避免自我欺骗
- 可与 [[并行化 Parallelization]] Voting 组合：多生成者竞争 + 评估者择优

## 局限与风险

- 多次 LLM 调用，成本和延迟显著增加
- 评估者本身可能有偏见或错误，导致错误优化方向
- 可能陷入局部最优：反复修改同一问题而非根本改进
- 需要设置最大循环次数，防止死循环

## 关键决策点

| 决策 | 建议 | 理由 |
|------|------|------|
| 评估者与生成者 | 使用不同 Prompt（甚至不同模型） | 避免自我欺骗，确保评估客观 |
| 评估标准 | 明确、可量化、可操作 | 模糊标准导致无效迭代 |
| 最大循环次数 | 3-5 次 | 超过后收益递减，成本递增 |
| 退出条件 | 评分达标 or 达到上限 | 取当前最佳，避免无限循环 |
| 反馈粒度 | 具体到可操作的修改建议 | 模糊反馈导致生成者无法有效改进 |

## 与其他模式的关系

- 与 [[并行化 Parallelization]] Voting 的关系：Voting 是"多生成取共识"，Evaluator-Optimizer 是"生成-反馈循环"，两者可组合
- 生成者内部可使用 [[Prompt Chaining]] 构建多步生成流程
- 可被 [[编排者-工人模式 Orchestrator-Worker]] 嵌入为质量保障阶段
- [[Human-in-the-Loop]] 可在评估者环节引入人工判断

## 参考资料
- [[构建模式总览]]
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
