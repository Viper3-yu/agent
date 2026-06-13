---
title: LlamaIndex
aliases: [LlamaIndex, llama_index, GPT Index]
tags:
  - framework
  - topic/rag
  - topic/data-agent
  - status/seed
created: 2026-06-10
updated: 2026-06-10
language: Python/TypeScript
version: ""
repo: https://github.com/run-llama/llama_index
license: MIT
---

# LlamaIndex

> LLM 驱动的数据智能框架。专注于将私有数据与 LLM 连接，提供数据连接、索引构建、查询引擎和 Agent 能力。核心理念：**Your Data, Your LLM**。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://www.llamaindex.ai |
| GitHub | https://github.com/run-llama/llama_index |
| 语言 | Python, TypeScript |
| 最新版本 | 0.12.x+ |
| 许可证 | MIT |
| GitHub Stars | 40k+ |

## 核心架构

LlamaIndex 的架构围绕 **数据摄入 -> 索引构建 -> 查询/对话** 的流水线设计：

```
┌─────────────────────────────────────────────────────────────┐
│                      LlamaIndex 架构                         │
│                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │  Data     │    │  Index   │    │  Query   │              │
│  │Connectors │──> │ Structure│──> │ Engine   │──> Response  │
│  │(LlamaHub) │    │          │    │          │              │
│  └──────────┘    └──────────┘    └──────────┘              │
│       │               │               │                     │
│       v               v               v                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │  150+    │    │  Vector  │    │  Chat    │              │
│  │ 数据源   │    │  Tree    │    │  Agent   │              │
│  │  连接器  │    │  KG ...  │    │  Tools   │              │
│  └──────────┘    └──────────┘    └──────────┘              │
└─────────────────────────────────────────────────────────────┘
```

### 核心组件

| 组件 | 功能 | 说明 |
|------|------|------|
| **Data Connectors** | 数据摄入 | 从 150+ 数据源加载文档（LlamaHub） |
| **Documents/Nodes** | 数据模型 | Document 是原始文档，Node 是文档的原子块 |
| **Index Structures** | 索引构建 | 多种索引结构支持不同检索模式 |
| **Query Engine** | 查询接口 | 无状态的问答接口 |
| **Chat Engine** | 对话接口 | 有状态的多轮对话接口 |
| **Agents** | 智能代理 | LLM 驱动的推理代理，可使用工具 |

## 快速上手

### 安装

```bash
# 核心库
pip install llama-index

# 或按需安装
pip install llama-index-core           # 核心
pip install llama-index-llms-openai    # OpenAI LLM
pip install llama-index-embeddings-openai  # OpenAI 嵌入
```

### 最小示例：RAG 查询

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader

# 1. 加载文档
documents = SimpleDirectoryReader("./data").load_data()

# 2. 构建索引
index = VectorStoreIndex.from_documents(documents)

# 3. 创建查询引擎
query_engine = index.as_query_engine()

# 4. 提问
response = query_engine.query("这份文档的主要内容是什么？")
print(response)
```

### 最小示例：多轮对话

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader

documents = SimpleDirectoryReader("./data").load_data()
index = VectorStoreIndex.from_documents(documents)

# 创建聊天引擎（有记忆）
chat_engine = index.as_chat_engine(chat_mode="condense_question")

# 多轮对话
response1 = chat_engine.chat("什么是机器学习？")
print(response1)

response2 = chat_engine.chat("它和深度学习有什么区别？")
print(response2)

# 重置对话历史
chat_engine.reset()
```

### 最小示例：Agent 工具调用

```python
from llama_index.core.agent import ReActAgent
from llama_index.llms.openai import OpenAI
from llama_index.core.tools import QueryEngineTool, ToolMetadata

llm = OpenAI(model="gpt-4o")

# 将索引包装为工具
query_tool = QueryEngineTool.from_defaults(
    query_engine=query_engine,
    metadata=ToolMetadata(
        name="document_search",
        description="搜索并查询内部文档中的信息"
    )
)

# 创建 Agent
agent = ReActAgent.from_tools(
    tools=[query_tool],
    llm=llm,
    verbose=True
)

# Agent 会自主决定是否使用工具
response = agent.chat("根据文档，公司的营收增长率是多少？")
print(response)
```

## 关键概念

### Data Connectors（数据连接器）

LlamaHub 提供 150+ 数据连接器：

```python
# 从各种数据源加载
from llama_index.readers.file import PDFReader
from llama_index.readers.notion import NotionReader
from llama_index.readers.slack import SlackReader
from llama_index.readers.web import SimpleWebPageReader

# PDF
pdf_docs = PDFReader().load_data(file="./report.pdf")

# 网页
web_docs = SimpleWebPageReader().load_data(urls=["https://example.com"])

# Notion
notion_docs = NotionReader().load_data(database_id="xxx")
```

### Index Structures（索引结构）

| 索引类型 | 检索方式 | 适用场景 |
|----------|---------|---------|
| **Vector Store Index** | 语义相似度 | 通用 RAG，最常用 |
| **Summary Index** | 顺序遍历 | 需要遍历所有文档的场景 |
| **Tree Index** | 层次摘要 | 文档层次结构明显时 |
| **Keyword Table Index** | 关键词匹配 | 精确关键词检索 |
| **Knowledge Graph Index** | 实体关系 | 结构化知识图谱 |
| **Document Summary Index** | 摘要匹配 | 大量文档的快速定位 |

```python
from llama_index.core import (
    VectorStoreIndex,
    SummaryIndex,
    TreeIndex,
    KeywordTableIndex,
    KnowledgeGraphIndex,
)

# Vector Store Index（最常用）
vector_index = VectorStoreIndex.from_documents(documents)

# Summary Index
summary_index = SummaryIndex.from_documents(documents)

# Tree Index
tree_index = TreeIndex.from_documents(documents)

# Keyword Table Index
keyword_index = KeywordTableIndex.from_documents(documents)
```

### Query Engine（查询引擎）

```python
# 基础查询
query_engine = index.as_query_engine(
    similarity_top_k=3,          # 返回最相关的 3 个节点
    response_mode="compact",     # 响应模式
)

# 带过滤的查询
from llama_index.core.vector_stores import MetadataFilters, FilterCondition

query_engine = index.as_query_engine(
    filters=MetadataFilters(
        filters=[
            {"key": "category", "value": "finance"},
            {"key": "year", "value": "2025"},
        ],
        condition=FilterCondition.AND,
    )
)

# 子问题查询引擎（复杂问题分解）
from llama_index.core.query_engine import SubQuestionQueryEngine
```

### Chat Engine（聊天引擎）

```python
# 不同聊天模式
chat_engine = index.as_chat_engine(chat_mode="best")           # 自动选择最佳模式
chat_engine = index.as_chat_engine(chat_mode="condense_question")  # 压缩历史问题
chat_engine = index.as_chat_engine(chat_mode="react")          # ReAct 推理
chat_engine = index.as_chat_engine(chat_mode="openai")         # OpenAI 兼容
```

### Agents（智能代理）

```python
from llama_index.core.agent import ReActAgent, OpenAIAgent
from llama_index.core.tools import FunctionTool

# 自定义工具
def search_database(query: str) -> str:
    """搜索公司内部数据库"""
    # 实现搜索逻辑
    return results

tool = FunctionTool.from_defaults(fn=search_database)

# ReAct Agent
agent = ReActAgent.from_tools(
    tools=[tool, query_engine_tool],
    llm=llm,
    verbose=True,
    max_iterations=10,
)

# 多轮对话
response = agent.chat("帮我查找去年的销售数据并总结趋势")
```

### LlamaParse（文档解析）

```python
from llama_parse import LlamaParse

# 高质量文档解析（支持表格、图表等复杂布局）
parser = LlamaParse(
    result_type="markdown",      # 输出格式
    num_workers=4,               # 并行处理数
    verbose=True,
)

documents = parser.load_data("./complex_report.pdf")
```

## 典型使用场景

- **企业知识库问答**：连接内部文档、Wiki、数据库，构建智能问答系统
- **文档智能分析**：PDF/Word/PPT 等文档的内容提取和分析
- **多数据源 RAG**：统一查询多个数据源（Confluence、Notion、Slack 等）
- **数据 Agent**：构建能自主使用工具查询数据的智能代理
- **研究助手**：论文搜索、文献综述、知识图谱构建
- **客户支持**：基于产品文档的自动问答系统

## 与同类对比

| 维度 | LlamaIndex | LangChain | Haystack | RAGatouille |
|------|------------|-----------|----------|-------------|
| 核心定位 | 数据连接 + RAG | 通用 LLM 应用 | 搜索 + NLP | 高级 RAG |
| 数据连接器 | 150+ (LlamaHub) | 丰富 | 丰富 | 有限 |
| 索引类型 | 6+ 种 | 依赖外部 | 多种 | ColBERT |
| Agent 能力 | 内置 | 内置 | 内置 | 无 |
| 学习曲线 | 低-中 | 中 | 中高 | 中 |
| TypeScript | 支持 (llama-ts) | 不支持 | 不支持 | 不支持 |
| 生产就绪 | 高 | 高 | 高 | 中 |

## 已知局限

- **文档碎片化**：过度拆分文档可能导致上下文丢失
- **嵌入质量依赖**：索引效果高度依赖嵌入模型的质量
- **大规模索引成本**：向量化大量文档需要显著的计算资源和 API 费用
- **复杂查询局限**：多跳推理和跨文档关联查询仍需改进
- **版本迭代快**：API 变化频繁，升级可能需要代码调整
- **中文支持**：默认分词和嵌入对中文优化不足，需选择合适的中文嵌入模型

## 参考资料

- [LlamaIndex 官方文档](https://docs.llamaindex.ai)
- [LlamaIndex GitHub](https://github.com/run-llama/llama_index)
- [LlamaHub 数据连接器](https://llamahub.ai)
- [LlamaIndex 博客](https://www.llamaindex.ai/blog)
- [LlamaParse 文档解析](https://github.com/run-llama/llama_parse)
