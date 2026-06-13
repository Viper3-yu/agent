---
title: Letta
aliases: [Letta, MemGPT]
tags:
  - framework
  - topic/agent
  - topic/memory
  - status/seed
created: 2026-06-12
updated: 2026-06-12
language: Python
version: "2026"
repo: https://github.com/letta-ai/letta
license: Apache 2.0
---

# Letta

> 前身 MemGPT，专注 Agent 长期记忆管理的框架。核心创新是让 Agent 拥有持久化、可编辑的长期记忆。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://www.letta.com |
| GitHub | https://github.com/letta-ai/letta |
| 语言 | Python |
| 最新版本 | 2026 |
| 许可证 | Apache 2.0 |

## 核心架构

核心创新是分层记忆管理：短期记忆（对话上下文）+ 长期记忆（持久化存储），Agent 可以主动编辑和搜索自己的记忆。

## 快速上手

### 安装

```bash
pip install letta
```

### 最小示例

```python
from letta import create_client

client = create_client()
agent = client.create_agent(
    name="my_agent",
    persona="I am a helpful assistant with long-term memory.",
)
response = client.send_message(agent.id, "Hello!")
```

## 关键概念

### 持久化 Agent 记忆

Agent 的记忆跨会话持久化，不会因对话结束而丢失。

### 记忆可编辑、可搜索

Agent 可以主动修改、删除、搜索自己的记忆条目。

### 状态管理

完整的 Agent 状态管理，包括记忆、配置、历史等。

## 典型使用场景

1. **长期记忆对话 Agent**: 需要记住历史上下文的对话系统
2. **个人助手**: 长期陪伴的 AI 助手，记住用户偏好
3. **知识积累**: Agent 在交互中不断积累和组织知识

## 与同类对比

| 特性 | Letta | LangGraph | CrewAI |
|------|-------|-----------|--------|
| 核心优势 | 长期记忆 | 图编排 | 角色协作 |
| 记忆管理 | 内置持久化 | 外部集成 | 有限 |
| 适用场景 | 记忆密集型 | 复杂工作流 | 团队协作 |

## 已知局限

- 专注记忆管理，编排能力有限
- 与其他框架的集成需要额外工作
- 记忆搜索的准确性依赖 embedding 质量

## 参考资料

- [Letta GitHub](https://github.com/letta-ai/letta)
