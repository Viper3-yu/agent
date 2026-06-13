---
title: Google ADK（Agent Development Kit）
aliases: [Google ADK, Agent Development Kit]
tags:
  - framework
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
language: Python, TypeScript, Java, Go, Kotlin
version: "2026"
repo: https://github.com/google/adk-python
license: Apache 2.0
---

# Google ADK（Agent Development Kit）

> Google 的多语言 Agent 框架，原生支持 A2A 协议，支持层次化多 Agent 编排。2026 年增长最快的 Agent 框架（~17.2k GitHub Stars）。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://google.github.io/adk-docs/ |
| GitHub | https://github.com/google/adk-python |
| 语言 | Python, TypeScript, Java, Go, Kotlin |
| 最新版本 | 2026 |
| 许可证 | Apache 2.0 |

## 核心架构

- **双 Agent 类型**：确定性工作流 Agent + 概率性 LLM Agent
- **图引擎**：有向图定义复杂工作流
- **A2A 原生**：自动生成 Agent Card
- **双向音视频流**：语音 Agent 支持

## 快速上手

### 安装

```bash
pip install google-adk
```

### 最小示例

```python
from google.adk import Agent

agent = Agent(
    name="my_agent",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant."
)
```

## 关键概念

### 核心特性

| 特性 | 说明 |
|------|------|
| 语言 | Python, TypeScript, Java, Go, Kotlin |
| 多 Agent | 层次化树 + 图工作流 |
| MCP 支持 | 通过适配器 |
| A2A 支持 | **原生**（最强） |
| 模型范围 | 模型无关（Gemini 优化） |
| 部署 | Vertex AI, Cloud Run, GKE |

### 确定性 vs 概率性 Agent

确定性工作流 Agent 执行固定流程，概率性 LLM Agent 由模型动态决策，两者可在同一图中混合使用。

### A2A 原生支持

自动生成 Agent Card，无需手动编写，是 A2A 生态中支持最完善的框架。

## 典型使用场景

1. **Google Cloud 用户**: Vertex AI / Cloud Run / GKE 部署
2. **多语言 Agent 开发**: Java/Go/Kotlin 等非 Python 场景
3. **企业级跨团队协作**: A2A 协议实现跨组织 Agent 互操作
4. **确定性+LLM 混合**: 严格区分确定性逻辑和 LLM 推理

## 与同类对比

| 特性 | Google ADK | Claude Agent SDK | OpenAI Agents SDK |
|------|-----------|-----------------|-------------------|
| 多语言 | 5 种语言 | 2 种 | 2 种 |
| A2A 支持 | 原生（最强） | 暂不支持 | 暂不支持 |
| 模型范围 | 模型无关 | 仅 Claude | 仅 OpenAI |
| 部署 | GCP 生态 | 自托管/API | OpenAI 平台 |

## 已知局限

- 文档和示例相对较少（框架较新）
- MCP 支持通过适配器，不如 Claude Agent SDK 原生
- 非 GCP 用户部署体验一般

## 参考资料

- [Google ADK 文档](https://google.github.io/adk-docs/)
- [Asher Tech 框架指南](https://ashertech.ca/blog/agentic-ai-tools-guide-2026)
- [TURION.AI 三方对比](https://turion.ai/blog/google-adk-vs-openai-claude-agent-sdk-2026/)
