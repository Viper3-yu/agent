---
title: Attention 机制
aliases: [Self-Attention, Multi-Head Attention, Scaled Dot-Product Attention]
tags:
  - concept
  - topic/llm
  - status/reviewed
created: 2026-06-12
updated: 2026-06-15
---

# Attention 机制

> 一种让模型在处理一个位置时，"加权关注"序列中所有相关位置的机制。是 [[Transformer 架构]] 的核心，也是所有现代 LLM 推理能力的来源。

## 核心思想

把序列中每个位置表示为**所有位置的加权和**，权重由"相关性"动态决定。

```
对于位置 i 的输出 y_i：

       ┌───── 来自位置 j 的贡献 ─────┐
y_i  =  Σ  α_ij · v_j      (j 遍历所有位置)
       j=1
              ↑
        权重 α_ij = f(q_i, k_j)  ← 由 query 和 key 算出
```

**直觉**：你在读"它"这个字时，会回头找"它"指代的是哪个名词——这就是 attention。

**三个角色**：
- **Query (Q)** —— 当前位置发出的"提问"（我想找什么？）
- **Key (K)** —— 每个位置的"标签"（我是什么？）
- **Value (V)** —— 每个位置的"内容"（我提供什么信息？）

## 工作原理

### 1. Scaled Dot-Product Attention

$$
\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V
$$

步骤：
1. **打分**：`score = Q · Kᵀ`，形状 (n, n)，第 (i, j) 个值是位置 i 对位置 j 的关注度
2. **缩放**：除以 √d_k，防止高维点积方差爆炸、softmax 饱和
3. **掩码**（可选）：Decoder 中用上三角 mask 屏蔽未来位置
4. **归一化**：softmax → 权重 α 矩阵（行和为 1）
5. **加权**：`α · V` → 每个位置的新表示

### 2. Self-Attention vs Cross-Attention

| 类型 | Q/K/V 来源 | 用途 |
|------|-----------|------|
| **Self-Attention** | 全部来自同一序列 | Encoder 内部、Decoder 第一子层 |
| **Cross-Attention** | Q 来自 Decoder，K/V 来自 Encoder | Decoder 第二子层、检索增强 |
| **Masked Self-Attention** | Self + 因果 mask | Decoder 自回归训练 |

### 3. Multi-Head Attention

把 Q/K/V 沿特征维度切 h 份，每份独立做 Attention，再 concat 回去：

```
MultiHead(Q,K,V) = Concat(head_1,...,head_h) · W^O
   head_i = Attention(Q·W_i^Q, K·W_i^K, V·W_i^V)
```

**为什么多头？** 不同 head 关注不同子空间（语法、指代、长程依赖等），相当于给模型一组"关注视角"。

### 4. 复杂度与显存

| 指标 | Self-Attention | RNN | Linear Attention |
|------|----------------|-----|------------------|
| 单层时间 | O(n² · d) | O(n · d²) | O(n · d²) |
| 单层空间 | O(n²) | O(n) | O(n) |
| 路径长度 | O(1) | O(n) | O(1) |
| 并行度 | ✅ 高 | ❌ 串行 | ✅ 高 |

**O(n²)** 是大痛点，催生了 [[FlashAttention]]、PagedAttention、Longformer、BigBird 等优化。

## 优势与局限

### ✅ 优势
- **O(1) 路径**：任意两位置直接相连，无信息衰减
- **可解释性**：注意力权重可视化，用于探查模型行为
- **统一接口**：Self / Cross / Masked / Sparse 同一套代码
- **可并行**：相比 RNN 是质变

### ❌ 局限
- **O(n²) 计算与显存**：长上下文瓶颈
- **无内置位置信息**：必须配合位置编码
- **Softmax 归一化**：让分布"全选一个"或"雨露均沾"，对长序列不利
- **KV Cache**：推理时每步重算所有 K/V，O(n) 增量

## 相关概念

- [[Transformer 架构]] — Attention 的载体
- [[MoE]] — 经常与 Multi-Head 联合优化
- [[Token 与上下文窗口]] — Attention 复杂度直接决定上下文长度上限
- [[../../04-Frameworks/MCP 协议/MCP 原理|MCP 原理]] — Cross-Attention 思想体现在检索增强
- [[../../04-Frameworks/AutoGen/AutoGen|AutoGen]] — 多 Agent 系统中 Attention-like 路由

## 参考资料

- Vaswani et al. "Attention Is All You Need" (NeurIPS 2017)
- Bahdanau et al. "Neural Machine Translation by Jointly Learning to Align and Translate" (2014) — 早期 Attention
- Lin et al. "A Structured Self-Attentive Sentence Embedding" (2017)
- The Illustrated Transformer — Jay Alammar
- Lilian Weng "Attention? Attention!" — https://lilianweng.github.io/posts/2018-06-24-attention/
