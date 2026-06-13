---
title: Reranking 模型
aliases: [Reranker, 重排序模型]
tags:
  - model
  - topic/rag
  - status/seed
created: 2026-06-12
updated: 2026-06-12
---

# Reranking 模型

> 对初步检索结果进行精细重排序的模型。Embedding 做粗筛，Reranker 做精排。

## 工作原理
```
查询 + 候选文档 → [Cross-Encoder Reranker] → 重新排序后的文档列表
```
Reranker 不是独立编码文档，而是将查询和文档一起输入，计算精确相关性分数。

## 主流 Reranker

| 模型 | 厂商 | 特点 |
|------|------|------|
| Cohere Rerank | Cohere | API 服务，效果最好 |
| bge-reranker-v2-m3 | BAAI | 开源最强 |
| Jina Reranker v2 | Jina | 开源，长文本支持 |
| flashrank | 开源 | 极速轻量 |

## 使用模式
```
Embedding 检索 top-50 → Reranker 精排 top-5 → 送入 LLM
```

## 参考资料
- [[Embedding 模型总览]]
- [[../01-Concepts/Agent 能力/RAG 高级技术|RAG 高级技术]]
