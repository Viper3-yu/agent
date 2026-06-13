---
title: Few-Shot Learning
aliases:
  - 少样本学习
tags:
  - prompt
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
---

# Few-Shot Learning

> 通过在 prompt 中提供少量示例引导模型行为

## 核心原理

Few-Shot Learning 利用大语言模型的 in-context learning 能力，通过在 prompt 中提供少量输入-输出示例，让模型理解任务模式并泛化到新的输入上。

## 示例设计要点

- **示例数量**：通常 2-5 个即可，过多示例会占用上下文窗口
- **示例多样性**：覆盖不同的输入模式和边界情况
- **示例质量**：确保示例准确且格式一致
- **顺序影响**：示例的排列顺序可能影响模型表现

## 与[[Chain of Thought]]的结合

将 Few-Shot 与 CoT 结合，即在示例中展示完整的推理过程（Few-shot CoT），能够进一步提升模型在复杂任务上的表现。

## 参考资料

- Brown et al., "Language Models are Few-Shot Learners", 2020
