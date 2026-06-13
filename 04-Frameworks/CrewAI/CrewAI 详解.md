---
title: CrewAI 详解
aliases: [CrewAI, Crew AI]
tags:
  - framework
  - topic/multi-agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-12
language: Python
version: ""
repo: https://github.com/crewAIInc/crewAI
license: MIT
---

# CrewAI 详解

> 基于角色的多 Agent 协作框架。52.4k GitHub Stars。用"船员"（Crew）概念组织 Agent 团队，每个 Agent 有角色、目标和背景故事。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://crewai.com |
| GitHub | https://github.com/crewAIInc/crewAI |
| 语言 | Python |
| 最新版本 | 0.108.0+ |
| 许可证 | MIT |
| GitHub Stars | 52.4k |

## 核心架构

CrewAI 的核心抽象模型是**角色扮演团队**，由四个关键组件构成：

| 组件 | 说明 |
|------|------|
| **Agent** | 有角色（Role）、目标（Goal）、背景故事（Backstory）的 AI 角色 |
| **Task** | 分配给 Agent 的具体任务，包含描述、期望输出和工具 |
| **Crew** | Agent 团队，定义协作策略（顺序/并行） |
| **Tool** | Agent 可使用的工具（搜索、代码执行、API 调用等） |

### 协作模式

- **顺序执行（Sequential）**：Agent 按顺序依次完成任务，前一个任务的输出作为后一个的输入
- **并行执行（Hierarchical）**：由 Manager Agent 协调多个 Agent 并行工作

### MCP 与 A2A 支持

- **MCP（Model Context Protocol）**：原生支持，Agent 可直接连接 MCP Server 使用工具
- **A2A（Agent-to-Agent）**：2026 年添加原生支持，实现跨框架 Agent 互操作

## 快速上手

### 安装

```bash
pip install crewai crewai-tools
```

### 最小示例

```python
from crewai import Agent, Task, Crew

# 定义 Agent
researcher = Agent(
    role="研究员",
    goal="深入研究给定主题并提供全面报告",
    backstory="你是一位经验丰富的研究分析师，擅长信息收集和分析。",
    verbose=True
)

writer = Agent(
    role="撰稿人",
    goal="将研究报告转化为引人入胜的文章",
    backstory="你是一位技术写作专家，擅长将复杂概念通俗化。",
    verbose=True
)

# 定义 Task
research_task = Task(
    description="研究 CrewAI 框架的最新发展和应用场景",
    expected_output="一份包含关键发现、趋势和见解的详细报告",
    agent=researcher
)

writing_task = Task(
    description="基于研究报告撰写一篇面向开发者的技术博客",
    expected_output="一篇结构清晰、通俗易懂的技术博客文章",
    agent=writer,
    context=[research_task]  # 依赖研究任务的输出
)

# 组建 Crew 并执行
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    verbose=True
)

result = crew.kickoff()
print(result)
```

## 关键概念

### Agent 配置

```python
from crewai import Agent
from crewai_tools import SerperDevTool

agent = Agent(
    role="数据分析师",
    goal="从数据中提取有价值的洞察",
    backstory="你在数据分析领域有10年经验。",
    tools=[SerperDevTool()],        # 可用工具列表
    llm="gpt-4o",                   # 指定 LLM
    max_iter=15,                    # 最大迭代次数
    allow_delegation=True,          # 允许委派任务给其他 Agent
    verbose=True
)
```

### Task 配置

```python
from crewai import Task

task = Task(
    description="分析销售数据并找出趋势",
    expected_output="包含图表和关键指标的分析报告",
    agent=analyst,
    tools=[data_analysis_tool],
    output_file="report.md",        # 输出到文件
    async_execution=False           # 是否异步执行
)
```

### 使用 YAML 配置

CrewAI 支持通过 YAML 文件定义 Agent 和 Task，适合大型项目：

```yaml
# config/agents.yaml
researcher:
  role: "资深研究分析师"
  goal: "发现最新的 AI 发展趋势"
  backstory: "你是一位在 AI 领域工作了15年的资深分析师"

# config/tasks.yaml
research_task:
  description: "调研 {topic} 的最新进展"
  expected_output: "一份详细的调研报告"
  agent: researcher
```

### MCP 工具集成

```python
from crewai import Agent, Crew, Task
from crewai_tools import MCPServerTool

# 连接 MCP Server
mcp_tool = MCPServerTool(
    server_url="http://localhost:3000",
    tool_name="database_query"
)

agent = Agent(
    role="数据查询专家",
    goal="查询并分析数据库中的数据",
    tools=[mcp_tool]
)
```

## 典型使用场景

- **快速原型验证**：快速搭建多 Agent 协作流程验证想法
- **内容生产流水线**：研究 -> 大纲 -> 撰写 -> 审校
- **代码开发流程**：需求分析 -> 架构设计 -> 编码 -> 测试
- **客户服务系统**：问题分类 -> 知识检索 -> 回答生成 -> 质量检查
- **数据分析管道**：数据收集 -> 清洗 -> 分析 -> 报告生成

## 与同类对比

| 维度 | CrewAI | LangGraph | AutoGen | OpenAI Agents SDK |
|------|--------|-----------|---------|-------------------|
| 核心模型 | 角色扮演团队 | 图状态机 | 对话协议 | 工具调用链 |
| 学习曲线 | 低 | 中高 | 中 | 低 |
| 灵活度 | 中 | 高 | 中 | 中 |
| MCP 支持 | 原生 | 需适配 | 需适配 | 原生 |
| A2A 支持 | 原生 | 无 | 无 | 无 |
| 适用场景 | 团队协作 | 复杂流程 | 多 Agent 对话 | 工具增强任务 |

## 已知局限

- **自定义 LLM 限制**：主要围绕 OpenAI API 设计，其他 LLM 集成需要额外配置
- **调试复杂度**：多 Agent 交互的调试和追踪不够直观
- **生产就绪性**：更偏向原型验证，大规模生产部署需要额外的基础设施
- **成本控制**：多 Agent 对话容易产生大量 Token 消耗
- **确定性不足**：相同的输入可能产生不同结果，难以保证一致性

## 参考资料

- [CrewAI 官方文档](https://docs.crewai.com)
- [CrewAI GitHub](https://github.com/crewAIInc/crewAI)
- [CrewAI Tools](https://github.com/crewAIInc/crewAI-tools)
- [CrewAI Flow 文档](https://docs.crewai.com/concepts/flows)
