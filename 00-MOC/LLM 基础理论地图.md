---
title: LLM 基础理论地图
aliases: [LLM Theory, 基础理论地图]
tags:
  - MOC
  - topic/llm
  - status/reviewed
created: 2026-06-13
updated: 2026-06-13
---

# LLM 基础理论地图

> 大语言模型**底层机制**的概念索引。从"模型怎么读懂文字"到"模型怎么压缩到能跑"。

## 入口
- → [[欢迎|返回 vault 首页]]
- → [[../00-MOC/KB 构建规范|KB 构建规范]]

## Token 与表示
- [[../01-Concepts/基础理论/Tokenization|Tokenization（分词）]] — BPE / WordPiece / Unigram / SentencePiece 四大算法
- [[../01-Concepts/基础理论/Embedding|Embedding（向量表示）]] — Word2Vec → BERT → CLIP 的演进 + MTEB 基准

## 模型架构
- [[../01-Concepts/基础理论/MoE|MoE（混合专家）]] — 路由 + 专家，2024 以来旗舰标配
- [[../01-Concepts/Agent 架构/Transformer 架构|Transformer 架构]]（位于 Agent 架构下，但属于基础理论）

## 推理时控制
- [[../01-Concepts/基础理论/采样策略|采样策略]] — Temperature / Top-k / Top-p / Min-p / Beam
- [[../01-Concepts/Agent 能力/对齐与安全|对齐与安全]]（在 Agent 能力下，但与采样紧密相关）

## 部署与优化
- [[../01-Concepts/基础理论/量化|量化]] — PTQ / QAT / GPTQ / AWQ / NF4 / GGUF
- → [[../06-Resources/微调/微调总览|微调总览]]（QLoRA = 量化 + LoRA）

## 训练范式
- [[../01-Concepts/Agent 能力/RLHF 与对齐方法|RLHF]]（占位，待写）
- [[../01-Concepts/Agent 能力/预训练范式|预训练]]（占位，待写）

## 相关 MOC
- → [[LLM 模型地图|模型层]]：上面这些理论如何在不同模型上落地
- → [[Agent 全景地图|应用层]]：上面这些理论如何支撑 Agent 系统
- → [[Prompt Engineering 地图|提示工程]]：调用模型时的技巧
- → [[项目看板|项目层]]：把理论用到自己的项目里
