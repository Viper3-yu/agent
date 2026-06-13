---
title: Chain of Thought
aliases:
  - CoT
  - 思维链
tags:
  - prompt
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
---

# Chain of Thought

> 通过"让我们一步步思考"引导模型展示推理过程

## 核心思想

Chain of Thought (CoT) 是一种提示词技术，通过引导大语言模型逐步展示其推理过程，而非直接输出最终答案。这种方法能够显著提升模型在复杂推理任务上的表现。

## 变体

### Zero-shot CoT

在 prompt 末尾添加"让我们一步步思考"（Let's think step by step），无需提供任何示例即可激发模型的推理能力。

### Few-shot CoT

在 prompt 中提供包含推理过程的示例，让模型模仿示例的推理方式进行回答。

### Auto-CoT

自动生成多样化的推理示例，减少人工设计 prompt 的工作量。

## 适用场景

- 数学推理和计算
- 逻辑推理问题
- 多步骤的复杂问题
- 需要解释推理过程的场景

## 与[[ReAct Prompting]]的关系

CoT 专注于推理过程的展示，而 ReAct 在此基础上加入了行动（Acting）环节，使模型能够与外部环境交互。两者可以结合使用。

## 参考资料

- Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models", 2022
