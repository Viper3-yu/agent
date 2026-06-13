---
title: RAG 框架总览
aliases: [RAG Frameworks]
tags:
  - framework
  - topic/rag
  - status/mature
created: 2026-06-12
updated: 2026-06-12
language: "多语言"
version: "2026"
repo: "N/A（索引页）"
license: "各框架不同"
---

# RAG 框架总览

> 专注 RAG 场景的框架和工具，提供文档处理、检索、生成的完整管线。

## 核心架构

RAG（Retrieval-Augmented Generation）的标准管线：

```
文档 → 解析 → 分块 → Embedding → 存储（向量数据库）
                                    ↓
查询 → Embedding → 检索 → Reranking → 上下文组装 → LLM 生成
```

## 框架对比

| 框架 | 定位 | 核心能力 | 适合 |
|------|------|---------|------|
| [[../../04-Frameworks/LlamaIndex/RAG 实践|LlamaIndex]] | 数据连接 + RAG | 文档加载、索引、查询引擎 | 数据密集型 RAG |
| LangChain | 通用 LLM 框架 | Chain、Retriever、Memory | 通用 RAG 应用 |
| Haystack | NLP 管线 | Pipeline、组件化 | 企业级搜索 |
| RAGFlow | 端到端 RAG | 文档解析、深度理解 | 复杂文档（PDF、表格） |
| Dify | 低代码平台 | 可视化 RAG 编排 | 快速搭建 |

## 关键概念

### 文档解析工具

| 工具 | 特点 |
|------|------|
| Unstructured | 通用文档解析（PDF、Word、HTML） |
| Docling (IBM) | 深度 PDF 理解 |
| Marker | PDF → Markdown |
| LlamaParse | LlamaIndex 官方解析器 |

### 分块策略

- 固定长度分块
- 语义分块
- 递归分块
- 文档结构分块

### 检索增强

- Reranking（重排序）
- 混合搜索（向量 + 关键词）
- 查询改写
- HyDE（假设性文档嵌入）

## 典型使用场景

1. **企业知识库**: 内部文档的智能问答
2. **客服系统**: 基于产品文档的自动回复
3. **代码助手**: 基于代码库的上下文检索
4. **研究助手**: 学术论文的检索和总结

## 选型建议

| 场景 | 推荐 |
|------|------|
| 数据密集型 | LlamaIndex |
| 通用应用 | LangChain |
| 复杂文档 | RAGFlow |
| 快速搭建 | Dify |
| 企业搜索 | Haystack |

## 已知局限

- 分块策略对检索质量影响大，需要调优
- Embedding 模型选择影响语义理解能力
- 长文档的上下文窗口限制
- 幻觉问题仍需 Reranking 和后处理缓解

## 参考资料

- [[../../01-Concepts/Agent 能力/RAG 高级技术|RAG 高级技术]]
- [[../../04-Frameworks/向量数据库/向量数据库总览|向量数据库]]
