---
title: RAG 高级技术
aliases: [Advanced RAG, 高级 RAG]
tags:
  - concept
  - topic/rag
  - status/mature
created: 2026-06-12
updated: 2026-06-12
---

# RAG 高级技术

> 超越基础 RAG（检索-拼接-生成）的进阶技术，解决检索质量、上下文利用、多跳推理等问题。

## 基础 RAG 回顾
→ [[RAG 检索增强生成]]

## 高级技术分类

### 检索优化
- **HyDE**（Hypothetical Document Embeddings）：先让 LLM 生成假设答案，用假设答案做检索，比原始查询更精准
- **Multi-Query Retrieval**：将原始查询改写为多个变体，分别检索后合并去重
- **Reranking**：初步检索后用 Cross-Encoder 重排序，提升相关性
- **Hybrid Search**：向量检索 + 关键词检索（BM25）混合使用

### 分块策略
- **固定大小分块**：按 token 数切分（简单但可能切断语义）
- **语义分块**：按语义边界切分（段落、章节）
- **递归分块**：先按大块切，再逐级细分
- **Parent-Child 分块**：小块用于检索，返回大块（父块）给 LLM 提供更多上下文

### 多跳推理
- **Multi-hop RAG**：第一轮检索的结果作为第二轮查询的输入，逐步深入
- **Graph RAG**：将文档组织为知识图谱，通过图遍历检索关联信息
- **Self-RAG**：LLM 自主决定是否需要检索、检索是否相关、生成是否被检索支持

### 上下文窗口优化
- **Contextual Compression**：压缩检索结果，只保留与查询相关的部分
- **Lost in the Middle**：重要信息放在上下文的开头或结尾（中间容易被忽略）

## 向量数据库
→ [[../04-Frameworks/向量数据库/向量数据库总览|向量数据库总览]]

## Embedding 模型
→ [[../02-Models/Embedding 模型/Embedding 模型总览|Embedding 模型]]

## 参考资料
- [LangChain RAG 教程](https://python.langchain.com/docs/tutorials/rag/)
