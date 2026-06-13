---
title: Embedding 模型总览
aliases: [Embedding, 向量模型, 文本嵌入]
tags:
  - model
  - topic/rag
  - status/mature
created: 2026-06-12
updated: 2026-06-12
---

# Embedding 模型总览

> 将文本转换为向量（数值数组）的模型，是 RAG 和语义搜索的基础。

## 主流 Embedding 模型

| 模型 | 厂商 | 维度 | 特点 |
|------|------|------|------|
| text-embedding-3-large | OpenAI | 3072 | 通用最强，支持维度缩减 |
| text-embedding-3-small | OpenAI | 1536 | 性价比高 |
| embed-v4 | Cohere | 1024 | 多语言优秀，支持压缩 |
| BGE-M3 | BAAI | 1024 | 开源最强，多语言 |
| Jina Embeddings v3 | Jina | 1024 | 开源，支持长文本 |
| nomic-embed-text | Nomic | 768 | 开源轻量 |

## 选型建议
- **闭源 API**：text-embedding-3-small（通用场景）
- **多语言**：embed-v4 或 BGE-M3
- **本地部署**：BGE-M3 或 Jina v3
- **极致性价比**：nomic-embed-text

## 评估指标
- MTEB 排行榜（Massive Text Embedding Benchmark）
- 检索准确率（Recall@K）
- 语义相似度（Cosine Similarity）

## 参考资料
- [MTEB Leaderboard](https://huggingface.co/spaces/mteb/leaderboard)
