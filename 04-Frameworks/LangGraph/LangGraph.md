---
title: LangGraph
aliases: [LangGraph]
tags:
  - framework
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-12
language: Python, TypeScript
version: "2026"
repo: https://github.com/langchain-ai/langgraph
license: MIT
---

# LangGraph

> LangChain 团队推出的有向图 Agent 框架。用图节点和边定义工作流，支持持久化状态和复杂分支逻辑。模型无关，是"不确定该选哪个时最安全的选择"。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://langchain-ai.github.io/langgraph/ |
| GitHub | https://github.com/langchain-ai/langgraph |
| 语言 | Python, TypeScript |
| 最新版本 | 2026 |
| 许可证 | MIT |

## 核心架构

以有向图为核心抽象，节点是处理步骤，边是连接和条件路由，图级别的共享状态贯穿整个执行过程。

## 快速上手

### 安装

```bash
pip install langgraph
```

### 最小示例

```python
from langgraph.graph import StateGraph

# 定义状态
class AgentState(TypedDict):
    messages: list

# 创建图
graph = StateGraph(AgentState)
graph.add_node("agent", call_model)
graph.add_node("tools", tool_node)
graph.add_edge("agent", "tools")
graph.add_conditional_edges("tools", should_continue)

app = graph.compile()
```

## 关键概念

### 节点

每个节点是一个处理步骤（LLM 调用、工具使用、条件判断）。

### 边

连接节点，支持条件路由，根据状态决定下一步走向。

### 状态

图级别的共享状态，节点可读写，贯穿整个执行过程。

### 检查点

内置持久化，支持中断恢复，可搭配 Temporal / Inngest 增强耐久性。

## 典型使用场景

1. **复杂有状态工作流**: 需要持久化状态的多步骤流程
2. **分支和循环**: 条件路由、循环执行、人工审批
3. **企业级可观测性**: 需要持久化和追踪的生产环境
4. **模型无关需求**: 不绑定特定 LLM 厂商

## 与同类对比

| 特性 | LangGraph | CrewAI | AutoGen |
|------|-----------|--------|---------|
| 抽象模型 | 有向图 | 角色+任务 | 对话 |
| 状态管理 | 内置持久化 | 有限 | 有限 |
| 模型无关 | ✅ | ✅ | ✅ |
| 学习曲线 | 中等 | 低 | 低 |

### 与 LangChain 的关系

LangChain = 通用 LLM 应用框架
LangGraph = 专注 Agent 编排的图引擎（LangChain 生态内）

## 已知局限

- 学习曲线较陡，图概念需要理解
- 简单场景下过度设计
- 调试复杂图时可观测性工具不足

## 参考资料

- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [Asher Tech 框架指南](https://ashertech.ca/blog/agentic-ai-tools-guide-2026)
