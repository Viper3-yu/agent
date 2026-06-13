---
title: Smolagents
aliases: [Smolagents, smol-agents, HuggingFace Agents]
tags:
  - framework
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-12
language: Python
version: "2026"
repo: https://github.com/huggingface/smolagents
license: Apache 2.0
---

# Smolagents

> HuggingFace 推出的轻量 Agent 框架，核心特色是代码生成型 Agent（直接生成 Python 代码执行，而非 JSON 工具调用）。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://huggingface.co/docs/smolagents |
| GitHub | https://github.com/huggingface/smolagents |
| 语言 | Python |
| 最新版本 | 2026 |
| 许可证 | Apache 2.0 |

## 核心架构

核心理念是"代码即动作"——Agent 直接生成 Python 代码来执行操作，而非输出 JSON 工具调用。这种方式更灵活，可表达复杂逻辑，但需要沙箱执行环境。

## 快速上手

### 安装

```bash
pip install smolagents
```

### 最小示例

```python
from smolagents import CodeAgent

agent = CodeAgent(tools=[], model="HuggingFaceTB/SmolLM2-1.7B-Instruct")
agent.run("Calculate the sum of 1 to 100")
```

## 关键概念

### 代码生成 vs 工具调用

传统 Agent 输出 JSON 工具调用，Smolagents 直接生成 Python 代码：
- 优势：更灵活，可表达复杂逻辑（循环、条件、变量）
- 风险：需要沙箱执行环境保障安全

### 核心特性

| 特性 | 说明 |
|------|------|
| 语言 | Python |
| 核心理念 | 代码即动作 |
| 多 Agent | 支持 |
| MCP 支持 | 已支持 |
| 模型范围 | 模型无关 |

## 典型使用场景

1. **数据科学 / 研究场景**: 需要灵活代码执行的数据分析
2. **快速原型**: 快速实验和验证 Agent 想法
3. **HuggingFace 生态**: 与 HF 模型和数据集深度集成

## 与同类对比

| 特性 | Smolagents | LangGraph | Pydantic AI |
|------|-----------|-----------|-------------|
| 核心方式 | 代码生成 | 图编排 | 类型安全 |
| 灵活性 | 高 | 中 | 低 |
| 安全性 | 需沙箱 | 高 | 高 |
| 学习曲线 | 低 | 中 | 低 |

## 已知局限

- 代码执行需要沙箱环境，部署复杂度高
- 生成的代码质量依赖模型能力
- 生产环境安全性需要额外保障

## 参考资料

- [Smolagents GitHub](https://github.com/huggingface/smolagents)
