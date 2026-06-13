---
title: Pydantic AI
aliases: [Pydantic AI, PydanticAI]
tags:
  - framework
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-12
language: Python
version: "2026"
repo: https://github.com/pydantic/pydantic-ai
license: MIT
---

# Pydantic AI

> 基于 Pydantic 的类型安全 Agent 框架，专注结构化输出。适合需要严格类型保障的场景。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://ai.pydantic.dev |
| GitHub | https://github.com/pydantic/pydantic-ai |
| 语言 | Python |
| 最新版本 | 2026 |
| 许可证 | MIT |

## 核心架构

基于 Pydantic 的类型系统，将 LLM 输出强制约束为定义好的 Pydantic 模型，确保结构化输出的可靠性和类型安全。

## 快速上手

### 安装

```bash
pip install pydantic-ai
```

### 最小示例

```python
from pydantic_ai import Agent

agent = Agent(
    'openai:gpt-4o',
    system_prompt='Be concise, reply with one sentence.',
)

result = agent.run_sync('Where does "hello world" come from?')
print(result.output)
```

## 关键概念

### 核心特性

| 特性 | 说明 |
|------|------|
| 语言 | Python |
| 核心理念 | 类型安全的结构化输出 |
| 多 Agent | 不支持 |
| MCP 支持 | 不支持 |
| 依赖 | Pydantic |

### 结构化输出

通过 Pydantic 模型定义输出 schema，LLM 返回的内容会被自动解析和验证。

## 典型使用场景

1. **类型安全的 LLM 集成**: 需要严格的输入/输出类型检查
2. **API 服务集成**: 在 FastAPI 等框架中集成 LLM
3. **简单 Agent 场景**: 不需要复杂编排，只需可靠的结构化输出

## 与同类对比

| 特性 | Pydantic AI | Smolagents | LangGraph |
|------|------------|-----------|-----------|
| 核心优势 | 类型安全 | 代码生成 | 图编排 |
| 复杂度 | 低 | 低 | 中 |
| 多 Agent | 不支持 | 支持 | 支持 |
| 适用规模 | 小型 | 中型 | 大型 |

## 已知局限

- 不支持多 Agent 编排
- 不支持 MCP 协议
- 功能相对单一，专注结构化输出

## 参考资料

- [Pydantic AI GitHub](https://github.com/pydantic/pydantic-ai)
