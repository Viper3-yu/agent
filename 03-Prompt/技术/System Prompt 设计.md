---
title: System Prompt 设计
aliases:
  - 系统提示词
tags:
  - prompt
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
---

# System Prompt 设计

> System Prompt 定义 Agent 的角色、能力和行为边界

## 设计原则

### Role（角色定义）
明确 Agent 的身份和专业领域，如"你是一个资深的 Python 开发者"。

### Context（上下文设定）
提供任务背景、用户信息、环境约束等上下文。

### Constraints（行为约束）
设定 Agent 的行为边界，包括禁止事项、安全限制、输出格式要求。

### Format（输出格式）
规定输出的结构和格式，如 JSON、Markdown、特定模板。

## 常见模式

- **角色扮演模式**：定义一个具体的角色身份
- **任务分解模式**：将复杂任务拆分为步骤
- **格式约束模式**：严格规定输出格式
- **安全防护模式**：设定安全边界和拒绝策略

## 与[[../../01-Concepts/Agent 架构/多智能体架构|多智能体]]的配合

在多智能体架构中，每个 Agent 的 System Prompt 需要明确定义其职责范围、协作方式和通信协议。

## 参考资料

- OpenAI, "GPT Best Practices"
