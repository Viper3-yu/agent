---
title: Self-Consistency
aliases:
  - 自一致性
tags:
  - prompt
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
---

# Self-Consistency

> 多次采样取一致性最高的答案，提高推理准确率

Self-Consistency 是一种解码策略，通过对同一个问题进行多次采样（使用较高的温度参数），然后选择出现频率最高的答案作为最终结果。这种方法基于一个假设：正确的推理路径通常比错误路径更多。

## 参考资料

- Wang et al., "Self-Consistency Improves Chain of Thought Reasoning in Language Models", 2023
