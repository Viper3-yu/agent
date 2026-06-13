---
title: ReAct Prompting
aliases:
  - ReAct Prompt
tags:
  - prompt
  - topic/agent
  - status/mature
created: 2026-06-12
updated: 2026-06-12
---


# ReAct Prompting

> 结合推理(Reasoning)与行动(Acting)的提示词模式

ReAct 是一种将推理与行动相结合的提示词框架，使模型能够在思考的同时执行外部操作。详见 [[../../01-Concepts/Agent 架构/ReAct 架构|ReAct 架构]]。

## 格式

ReAct 的核心是 Thought/Action/Observation 循环：

1. **Thought**：模型对当前状态进行分析和推理
2. **Action**：基于推理结果执行一个具体操作（如搜索、查询、计算）
3. **Observation**：获取操作结果
4. 重复以上循环直到得出最终答案

## 示例

```
Question: 哥伦比亚的行政首都和最大城市分别是哪里？

Thought 1: 我需要分别查找哥伦比亚的行政首都和最大城市。
Action 1: Search[哥伦比亚行政首都]
Observation 1: 波哥大（Bogotá）是哥伦比亚的首都和行政中心。

Thought 2: 波哥大是行政首都，现在我需要确认最大城市。
Action 2: Search[哥伦比亚最大城市]
Observation 2: 波哥大也是哥伦比亚最大的城市。

Thought 3: 两者都是波哥大。
Action 3: Finish[行政首都和最大城市都是波哥大]
```

## 参考资料

- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models", 2023
