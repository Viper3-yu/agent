---
title: Tree of Thought
aliases:
  - ToT
  - 思维树
tags:
  - prompt
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
---

# Tree of Thought

> 将推理过程组织为树状结构，允许回溯和探索多条路径

## 与Chain of Thought的区别

- **CoT** 是线性的单路径推理，一旦走错无法回头
- **ToT** 将推理过程组织为树状结构，每个节点是一个中间思考状态
- ToT 允许模型在多个候选路径中进行评估和选择
- ToT 支持回溯，当某条路径不可行时可以回退尝试其他路径

## 参考资料

- Yao et al., "Tree of Thoughts: Deliberate Problem Solving with Large Language Models", 2023
