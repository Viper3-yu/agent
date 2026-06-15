---
title: Transformer 架构
aliases: [Transformer, vanilla Transformer]
tags:
  - concept
  - topic/llm
  - status/reviewed
created: 2026-06-12
updated: 2026-06-15
---

# Transformer 架构

> 2017 年 Google 在 "Attention Is All You Need" 中提出的神经网络架构，以 Self-Attention 替代 RNN/CNN，成为所有现代 LLM 的基础骨架。

## 核心思想

**抛弃循环与卷积**，完全依赖 **Attention 机制**建模序列中任意两个位置的依赖关系。

```
输入: [x1, x2, ..., xn]  (token embeddings + positional encoding)
   ↓
Encoder 堆叠 ×N（默认 6 层）
   ↓
[context-aware representations]
   ↓
Decoder 堆叠 ×N（默认 6 层，含 Masked Self-Attention）
   ↓
输出: [y1, y2, ..., ym]  (线性 + Softmax)
```

三个关键设计：
1. **Self-Attention** —— O(1) 路径长度捕获长距离依赖（vs RNN 的 O(n)）
2. **Multi-Head** —— 多组 Q/K/V 并行学不同子空间
3. **位置编码**（sin/cos 或 RoPE）—— 注入序列顺序信息（Attention 本身是 permutation-invariant）

## 工作原理

### 单层 Encoder 块

```
x ──LayerNorm──► Multi-Head Self-Attention ──+(residual)
                                                │
                                                ▼
              ──LayerNorm──► Feed-Forward (FFN) ──+(residual)
                                                  │
                                                  ▼
                                                  output
```

### Scaled Dot-Product Attention

```
Attention(Q, K, V) = softmax(QKᵀ / √d_k) V
```

- `d_k` 是 Key 维度，√d_k 用于抵消高维点积方差爆炸
- **复杂度**：每层 O(n² · d)，序列长 n 时二次方
- **并行度**：相对 RNN 可整序列并行，是 LLM 训练可行的核心

### Multi-Head Attention

```
MultiHead(Q, K, V) = Concat(head_1, ..., head_h) W^O
   where head_i = Attention(Q W_i^Q, K W_i^K, V W_i^V)
```

通常 `h=8` 或 `16`，每头 d_k = d_model / h，**总参数与单头相同**但子空间更丰富。

### Decoder 块（与 Encoder 的两个区别）

1. **Masked Self-Attention** —— 因果遮罩，第 i 位只能看 ≤ i 的位置（训练时用下三角 mask）
2. **Cross-Attention** —— Q 来自 Decoder，K/V 来自 Encoder 输出

### 位置编码

| 方案 | 代表模型 | 优缺点 |
|------|---------|--------|
| 固定 sin/cos | 原 Transformer | 简单、不可学习、长度外推差 |
| 可学习绝对 | BERT、GPT-2 | 灵活但长度受限 |
| **RoPE**（旋转） | LLaMA / Qwen / Mistral | 相对位置 + 长度外推好 |
| **ALiBi** | BLOOM | 简单、加偏置即可 |
| **YaRN** | Qwen 2.5+ | RoPE 扩展，支持 1M+ 上下文 |

## 优势与局限

### ✅ 优势
- **并行训练**：O(1) 路径长度，整序列并行（vs RNN 必须串行）
- **长距离依赖**：任意两位置直接相连，无信息瓶颈
- **可堆叠**：纯残差 + LayerNorm，深 100+ 层仍可训练
- **统一架构**：同一套结构适用 NLP / 视觉 / 语音 / 多模态

### ❌ 局限
- **O(n²) 复杂度**：长序列（n > 8K）显存爆炸 → 催生 FlashAttention / Linear Attention / 稀疏 Attention
- **位置信息需额外注入**：原生架构对顺序不敏感
- **KV Cache**：推理时 O(n) 缓存，长上下文仍是工程难题
- **训练数据饥渴**：参数大、需海量文本，无监督预训练才发挥

## 相关概念

- [[Attention 机制]] — Self-Attention 是其特例
- [[Tokenization]] — Transformer 之前的预处理
- [[MoE]] — 现代 LLM 常用 Transformer + MoE 稀疏化
- [[Token 与上下文窗口]] — 位置编码与上下文长度直接相关
- [[../../01-Concepts/Agent 架构/ReAct 架构|ReAct 架构]] — LLM 应用层架构（Agent 视角）

## 参考资料

- Vaswani et al. **"Attention Is All You Need"** (NeurIPS 2017) — https://arxiv.org/abs/1706.03762
- Su et al. **"RoFormer: Enhanced Transformer with Rotary Position Embedding"** (2021) — RoPE 论文
- The Illustrated Transformer — Jay Alammar 经典图解
- Karpathy **"Let's build GPT: from scratch, in code, spelled out"** — 配套视频

## 时间线

| 年 | 里程碑 |
|---|--------|
| 2017 | Transformer 论文，机器翻译 SOTA |
| 2018 | GPT-1 / BERT 验证预训练范式 |
| 2019-2020 | GPT-2/3、RoBERTa、T5 — 规模化 |
| 2021 | Vision Transformer (ViT) 跨界 |
| 2023+ | MoE + 长上下文 + 推理增强（o1 类） |
