---
title: 编排者-工人模式 Orchestrator-Worker
aliases: [Orchestrator-Worker, 编排模式]
tags:
  - concept
  - topic/agent
  - status/mature
category: workflow
complexity: medium
source: "Anthropic: Building Effective Agents"
created: 2026-06-12
updated: 2026-06-12
---

# 编排者-工人模式 Orchestrator-Worker

> 一个中央 Agent（编排者）负责理解任务、制定计划、分配子任务给工人 Agent，最后整合结果。

## 模式定义

Orchestrator-Worker 模式由一个中央编排者 Agent 动态分析任务、制定执行计划，并将子任务分配给多个工人 Agent 并行或顺序执行。与 [[Prompt Chaining]] 的固定流程不同，编排者根据任务复杂度动态决定需要多少个工人、每个工人做什么，最后整合所有工人的结果。这是大多数多 Agent 框架的核心模式。

## 适用场景

1. 任务复杂度不可预知，需要动态规划
2. 需要动态决定执行策略（工人数量、任务分配）
3. 多 Agent 框架（CrewAI、LangGraph）的核心编排需求
4. 大规模代码重构（编排者分析依赖，分配模块给工人）
5. 多步骤研究报告生成（编排者规划章节，工人并行撰写）

## 不适用场景

1. 任务流程固定且可预知 — [[Prompt Chaining]] 更简单高效
2. 单步任务 — 直接调用 API，无需编排开销
3. 对延迟极度敏感 — 编排者决策本身增加一次 LLM 调用

## 架构图

```
用户请求
    ↓
[编排者 Agent]
    ├── 1. 分析任务复杂度
    ├── 2. 制定执行计划
    ├── 3. 分配子任务
    │       ↓
    │   [工人A] → 结果A ─┐
    │   [工人B] → 结果B ─┤
    │   [工人C] → 结果C ─┘
    └── 4. 整合结果 → 最终输出
```

## 核心流程

1. 编排者接收用户请求，分析任务结构
2. 编排者制定执行计划，决定需要哪些工人及各自任务
3. 将子任务分配给工人 Agent（可并行或顺序）
4. 工人各自执行子任务，返回结果
5. 编排者整合所有工人结果，生成最终输出

## 伪代码示例

```python
def orchestrator_worker(user_request):
    # 编排者：分析任务并制定计划
    plan = call_llm(orchestrator_prompt, user_request)

    # 分配子任务给工人
    worker_tasks = []
    for subtask in plan.subtasks:
        worker_tasks.append(execute_worker(subtask))

    # 并行或顺序执行工人任务
    results = await asyncio.gather(*worker_tasks)

    # 编排者：整合结果
    final_output = call_llm(integration_prompt, results)
    return final_output

def execute_worker(subtask):
    """工人 Agent：执行分配的子任务"""
    return call_llm(worker_prompt, subtask.description, subtask.context)
```

## 优势

- 动态规划能力：编排者根据任务复杂度自适应调整策略
- 灵活性高：工人数量和任务分配可动态变化
- 可组合性强：工人内部可以使用 [[Prompt Chaining]] 或 [[并行化 Parallelization]]
- 是 LangGraph、CrewAI 等框架的核心模式，生态支持好

## 局限与风险

- 编排者决策质量直接影响整体效果（单点瓶颈）
- 额外的编排者 LLM 调用增加延迟和成本
- 编排者可能过度拆分简单任务，导致不必要的开销
- 工人之间的依赖关系需要编排者正确识别

## 关键决策点

| 决策 | 建议 | 理由 |
|------|------|------|
| 编排者模型 | 使用强推理模型 | 编排决策质量是关键 |
| 工人模型 | 根据子任务复杂度选择 | 简单任务用小模型，复杂任务用强模型 |
| 工人数量 | 3-7 个 | 过多增加编排复杂度和整合难度 |
| 执行方式 | 无依赖则并行，有依赖则顺序 | 最大化并行性，同时保证正确性 |
| 整合策略 | 结构化输出 + 模板合并 | 减少整合阶段的推理负担 |

## 框架映射

| 框架 | 编排者 | 工人 |
|------|--------|------|
| CrewAI | Manager Agent | Role Agent |
| LangGraph | Router Node | Worker Node |
| Claude Code | Main Agent | Subagent |

## 与其他模式的关系

- 与 [[Prompt Chaining]] 的区别：Chaining 固定流程顺序执行，Orchestrator-Worker 动态规划
- 可调用 [[Prompt Chaining]] 作为工人的内部执行逻辑
- 可使用 [[并行化 Parallelization]] 并行分配子任务给多个工人
- 可结合 [[路由模式 Routing]]：编排者先路由再分配
- [[评估者-优化者模式 Evaluator-Optimizer]] 可嵌入为编排者的一个阶段
- [[Human-in-the-Loop]] 可在编排者决策节点引入人工审批

## 参考资料
- [[构建模式总览]]
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
