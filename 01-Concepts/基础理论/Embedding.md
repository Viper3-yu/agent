---
title: Embedding（向量表示）
aliases: [Embedding, 向量表示, 嵌入]
tags:
  - concept
  - topic/llm
  - status/reviewed
created: 2026-06-13
updated: 2026-06-13
---

# Embedding（向量表示）

> 把离散的符号（词、句、图、图像、视频）映射到**稠密连续向量空间**，让语义相近的输入在向量空间也相近。这是所有现代 AI 的基石。

## 核心思想

```
"king"  → [0.2, -0.5, 0.8, ..., 0.3]   (768 维向量)
"queen" → [0.3, -0.4, 0.7, ..., 0.2]   ← 与 king 接近
"banana"→ [-0.8, 0.6, -0.1, ..., 0.5]  ← 与 king 远离
```

**关键性质**：
- **语义几何**：语义关系 = 向量运算
  - `vec("king") - vec("man") + vec("woman") ≈ vec("queen")`
- **距离度量**：可用余弦、欧氏距离算语义相似度
- **下游任务友好**：聚类、检索、分类可直接套用

## 工作原理

### 1. 文本 Embedding

#### Word2Vec（2013，静态词向量）

两种训练目标：
- **CBOW**：用上下文预测中心词
- **Skip-gram**：用中心词预测上下文

```
训练样本（Skip-gram）：中心词="fox"
  上下文窗口 ±2：["quick", "brown", "jumps", "over"]
  优化目标：最大化 P(quick|brown|fox) × P(brown|fox) × ...
```

局限：一个词只有一个向量，**无法处理一词多义**（"bank" 河岸 vs 银行）。

#### BERT Embedding（2018，上下文相关）

BERT 输出每个 token 的**上下文相关**向量。同一个 "bank" 在不同句子中向量不同：

```
"I deposited money in the bank"   → vec₁
"The river bank is muddy"         → vec₂ ≠ vec₁
```

代价：每次推理都要跑整个模型，**不能预计算**。

#### Sentence / Document Embedding

把整段文本压成一个向量。代表模型：

| 模型 | 维度 | 训练目标 | 特点 |
|------|------|---------|------|
| Sentence-BERT | 768 | NLI + 相似度 | 经典 baseline |
| OpenAI text-embedding-3-small | 1536 | 未知（专有） | 通用强 |
| OpenAI text-embedding-3-large | 3072 | 未知 | 长文本最强 |
| BGE / BGE-M3 | 1024 | C-MTEB 多任务 | 中文友好，开源 |
| MTEB Leader | 1024 | 多任务 | 2025 SOTA |
| Cohere embed-v3 | 1024 | 专有 | 多语言 + 压缩 |

### 2. 多模态 Embedding

#### CLIP（OpenAI 2021）

联合训练图像和文本 embedding 到**同一空间**：

```
"a photo of a cat" → vec_text ─┐
                                ├─ cos 距离 ≈ 0.8  → 匹配
cat.jpg          → vec_image  ─┘
"a photo of a car" → vec_text → cos ≈ 0.1 → 不匹配
```

应用：
- 零样本图像分类
- 文搜图 / 图搜文
- 图像检索系统

#### ImageBind（Meta 2023）

把 6 种模态（图像、文本、音频、深度、热成像、IMU）映射到同一空间，任两模态可互搜。

## 相似度度量

### 余弦相似度（最常用）

$$\cos(\mathbf{A}, \mathbf{B}) = \frac{\mathbf{A} \cdot \mathbf{B}}{\|\mathbf{A}\| \|\mathbf{B}\|}$$

- 范围：[-1, 1]
- 不受向量长度影响
- RAG 检索的事实标准

### 欧氏距离

$$d(\mathbf{A}, \mathbf{B}) = \|\mathbf{A} - \mathbf{B}\|_2$$

- 受向量范数影响
- 适合聚类（K-Means）

### 点积

$$\mathbf{A} \cdot \mathbf{B}$$

- 当向量已 L2 归一化时，等价于余弦相似度
- 计算最快，向量数据库首选

## 训练目标

### 对比学习（主流）

InfoNCE 损失：拉近正样本对、推远负样本对。

```
样本：("query", "positive_doc", 99 个 negative_docs)
损失 = -log( exp(sim(q, p)/τ) / Σ exp(sim(q, x_i)/τ) )
其中 τ 是温度参数（通常 0.05-0.1）
```

需要大量**负样本**——通常用 in-batch negatives（同一 batch 的其他样本作为负例）。

### 三种训练范式

| 范式 | 训练信号 | 适用 |
|------|---------|------|
| 有监督 | 人工标注的相似对 | 最高质量 |
| 弱监督 | 搜索点击、引用关系 | 大规模数据 |
| 自监督 | 同句正例、其他句负例 | 无标注数据 |

## 关键基准（MTEB）

[MTEB](https://huggingface.co/spaces/mteb/leaderboard) 是 Embedding 模型的"大考"，覆盖 8 大类任务：

| 任务类型 | 代表数据集 | 评估指标 |
|---------|----------|---------|
| 分类 | AmazonReviews | Accuracy |
| 聚类 | BiorxivClustering | V-measure |
| 检索 | ArguAna | nDCG@10 |
| 重排 | AskUbuntuRetrieval | MAP |
| STS | STS12 | Spearman |
| Summarization | SummEval | Spearman |
| Pair Classification | SprintDuplicateQuestions | AP |
| Bitext Mining | BUCC | F1 |

**平均得分（2026-06）**：
- OpenAI text-embedding-3-large: ~66.6
- BGE-M3: ~63.5
- MTEB Leader: ~68.2

## 与 RAG、向量数据库的关系

```
Query → Embedding → 向量数据库 ANN 检索 → Top-K 文档 → LLM 生成答案
            ↑                                            ↑
        文本 Embedding 模型                    文档 Embedding（离线预计算）
```

详见：
- [[RAG 检索增强生成]]——RAG 框架
- [[../02-Models/Embedding 模型/Embedding 模型总览]]——选型
- [[../02-Models/Embedding 模型/Reranking 模型]]——二阶段检索
- [[../04-Frameworks/向量数据库/向量数据库总览]]——向量数据库
- [[../04-Frameworks/RAG 框架/RAG 框架总览]]——RAG 框架

## 优势与局限

### 优势
- **统一语义空间**：让任意模态互相检索
- **预计算友好**：文档 embedding 可离线缓存
- **跨语言**：CLIP/BGE 等支持中英对齐
- **零样本**：无需训练即可用于检索/分类

### 局限
- **黑盒**：无法解释为什么两个向量相似
- **维度诅咒**：高维空间中距离度量失效（curse of dimensionality）
- **训练偏差**：模型偏见会进入 embedding（如性别刻板印象）
- **长文本退化**：>8K token 的文档，直接平均 embedding 质量差，需要分段
- **冷启动**：新词/新概念需要重新训练或用 LLM 重新生成 embedding

## 相关概念

- [[Tokenization]]——Embedding 的输入准备
- [[Transformer 架构]]——Embedding 层的来源
- [[Attention 机制]]——动态上下文相关的 Embedding
- [[RAG 检索增强生成]]——主要下游应用

## 参考资料

- [Word2Vec 论文](https://arxiv.org/abs/1301.3781)
- [BERT 论文](https://arxiv.org/abs/1810.04805)
- [CLIP 论文](https://arxiv.org/abs/2103.00020)
- [Sentence-BERT](https://www.sbert.net/)
- [MTEB Leaderboard](https://huggingface.co/spaces/mteb/leaderboard)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)