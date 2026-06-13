---
title: Prompt Chaining
aliases: [提示链, Sequential Chaining]
tags:
  - concept
  - topic/agent
  - status/seed
category: workflow
complexity: low
source: "Anthropic: Building Effective Agents"
created: 2026-06-12
updated: 2026-06-12
---

# Prompt Chaining

> 将复杂任务分解为多个简单步骤，每个步骤的输出作为下一步的输入。最基础也最可靠的 Agent 模式。

## 模式定义

Prompt Chaining 将一个复杂任务拆解为一系列顺序执行的子任务，每个子任务由独立的 Prompt 处理，前一步的输出作为后一步的输入。通过在关键步骤之间插入 gate check（门控验证），可以在中间环节检测错误并决定是否继续执行，从而提升整体可靠性。

## 适用场景

1. 每个步骤有明确的子目标
2. 需要中间验证（gate checks）
3. 顺序依赖的处理流程
4. 多步文档处理（提取 → 分析 → 总结）
5. 数据清洗管道（校验 → 转换 → 格式化）

## 不适用场景

1. 步骤之间无顺序依赖 — 应使用 [[并行化 Parallelization]]
2. 需要动态决定执行哪些步骤 — 应使用 [[编排者-工人模式 Orchestrator-Worker]]
3. 单步即可完成的简单任务 — 直接调用 API 即可

## 架构图

```
输入 → [步骤1: 提取关键信息] → 中间结果1
                              ↓
         [Gate Check: 验证格式] ← 失败 → 重试/终止
                              ↓ 通过
         [步骤2: 分析推理]     → 中间结果2
                              ↓
         [Gate Check: 验证逻辑] ← 失败 → 重试/终止
                              ↓ 通过
         [步骤3: 生成输出]     → 最终结果
```

## 核心流程

1. 将复杂任务分解为 N 个顺序子步骤
2. 为每个步骤编写独立的、聚焦的 Prompt
3. 在关键步骤之间插入 gate check，验证中间结果
4. 将前一步输出格式化为后一步的输入
5. 失败时重试当前步骤，而非全部重来

## 伪代码示例

```python
def prompt_chain(input_text, steps):
    result = input_text
    for step in steps:
        prompt = step.build_prompt(result)
        result = call_llm(prompt)

        # Gate check: 在关键步骤验证中间结果
        if step.has_gate_check:
            validation = call_llm(step.gate_prompt(result))
            if not validation.passed:
                if step.can_retry:
                    result = call_llm(prompt)  # 重试当前步骤
                else:
                    raise ChainFailedError(step.name, validation.reason)

    return result
```

## 优势

- 每个步骤简单明确，易于调试和优化
- Gate check 提供中间验证，提升可靠性
- 单步失败可重试，不需要全部重来
- 与裸 API 兼容，无需框架支持
- 流程透明，易于理解和维护

## 局限与风险

- 顺序执行带来累积延迟（每步都要等待前一步完成）
- 错误可能在早期步骤引入，后续步骤基于错误输入继续执行
- 固定流程缺乏灵活性，无法动态调整步骤

## 关键决策点

| 决策 | 建议 | 理由 |
|------|------|------|
| 步骤粒度 | 每步一个明确子目标 | 过粗难以调试，过细增加延迟 |
| Gate check 位置 | 关键转换点 | 平衡可靠性与延迟开销 |
| 重试策略 | 当前步骤重试 2-3 次 | 避免无限重试，超过阈值则终止 |
| 输出格式 | 步骤间使用结构化格式（JSON） | 减少解析错误 |

## 与其他模式的关系

- 比 [[路由模式 Routing]] 更线性，适合固定流程
- 可被 [[编排者-工人模式 Orchestrator-Worker]] 调用作为子任务
- 是 [[评估者-优化者模式 Evaluator-Optimizer]] 中生成链的基础
- 与 [[并行化 Parallelization]] 可组合：链内步骤可并行化

## 参考资料
- [[构建模式总览]]
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
