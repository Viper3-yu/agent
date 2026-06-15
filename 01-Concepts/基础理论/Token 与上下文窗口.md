---
title: Token 与上下文窗口
aliases: [Context Window, Token Limits, 上下文长度]
tags:
  - concept
  - topic/llm
  - status/reviewed
created: 2026-06-12
updated: 2026-06-15
---

# Token 与上下文窗口

> 上下文窗口是 LLM 一次能"看到"的最大 token 序列长度，决定了模型能处理多长的输入 / 多长的对话 / 多复杂的 Agent 任务。

## 核心思想

**Token** 是 [[Tokenization|分词]]后的最小处理单元，**不是字符也不是词**。一个 token 通常对应 0.75 个英文单词或 1.5~2 个中文字符（取决于分词器）。

**上下文窗口** = 输入 token 数 + 输出 token 数。两者共享同一窗口：

```
┌──────────────────────────────────┬───────────────────┐
│         Input (prompt)           │   Output (reply)  │
│         ←←←←←←←←←←→→→→→→→→→→→   │                   │
└──────────────────────────────────┴───────────────────┘
├─── 模型一次能处理的最大 token 总和 ───┤
```

**典型上下文长度对比（2026）**：

| 档位 | 长度 | 代表模型 |
|------|-----|---------|
| 短 | 8K-32K | 早期 GPT-3.5、Llama 2 |
| 中 | 128K-200K | Claude 3.5/3.7 Sonnet、GPT-4o、Qwen 2.5 |
| 长 | 400K-1M | Claude 3/4 Sonnet、Gemini 2.0、Qwen 2.5-1M |
| 超长 | 2M-10M | Gemini 3.1 Pro（2M）、Grok 4.20（2M）、Magic（100M 实验性） |

## 工作原理

### 1. 窗口的物理限制

**训练长度**：模型在预训练阶段见过的最大序列长度。外推超过训练长度通常会显著掉点。

**位置编码决定外推能力**：

| 编码方式 | 外推性能 | 代表 |
|---------|---------|------|
| 绝对位置编码（learnable） | ❌ 差 | BERT、GPT-2 |
| RoPE | ✅ 中 | LLaMA、Qwen |
| RoPE + YaRN 扩展 | ✅ 好 | Qwen 2.5-1M |
| ALiBi | ✅ 好 | BLOOM |
| NoPE（无显式位置） | ✅ 中 | 某些研究模型 |

### 2. 推理时的成本结构

**预填充（Prefill）**：处理整段输入 prompt，**并行计算**，O(n²) attention。
**解码（Decode）**：逐 token 自回归生成，**串行**，每步重算 attention。

**KV Cache**：把之前 token 的 K/V 缓存起来避免重算。**显存占用 = O(n · d · L)**，长上下文杀手。

```
n = 序列长度
d = 隐藏维度
L = 层数

8K 上下文、80 层、d=8192 的模型：
  KV cache ≈ 8K × 8192 × 80 × 2 (K+V) × 2 bytes (FP16)
          ≈ 20 GB    ← 单请求就要这么多
```

### 3. 长上下文优化技术

| 技术 | 核心思想 | 代表 |
|------|---------|------|
| **FlashAttention** | 显存换算力，IO-aware | Tri Dao 2022 |
| **PagedAttention** | 分页管理 KV cache | vLLM |
| **Speculative Decoding** | 小模型 draft + 大模型 verify | 各种 |
| **Ring Attention** | 多机分摊 attention | Liu et al. 2023 |
| **Mamba / SSM** | O(n) 复杂度 | 状态空间模型替代 Transformer |
| **YaRN** | RoPE 频率缩放 | Qwen 2.5-1M |
| **InfLLM** | 训练无关的 KV 压缩 | 清华 |

### 4. 上下文与"有效上下文"

**Lost-in-the-Middle**：研究表明 LLM 在长上下文中**对中间位置**的关注度显著低于首尾（Liu et al. 2023）。"上下文 1M" ≠ "模型能有效利用 1M"。

**有效上下文测试**：NIAH (Needle in a Haystack) 评测。

## 优势与局限

### ✅ 优势
- **单次输入更长** = 整本书、长代码库、长视频帧
- **Agent 任务更复杂** = 多工具调用、长对话历史、多步推理
- **RAG 替代** = 某些场景下不需外挂检索，全装进 prompt

### ❌ 局限
- **成本平方增长**：attention 是 O(n²)，128K 是 8K 的 256 倍计算
- **KV Cache 显存**：高并发场景长上下文 = 显存爆炸
- **Lost-in-the-Middle**：再长也未必"用得到"
- **位置编码外推**：超训练长度掉点
- **评测作弊**：RAG benchmark 难做，可能数据污染

## 上下文长度选型指南

| 场景 | 推荐长度 | 理由 |
|------|---------|------|
| 普通对话 | 32K-128K | 够用、便宜 |
| 长文档总结 | 200K-1M | 一本书约 200K token |
| 代码库 RAG | 200K+ | 整个中小项目 |
| 视频理解 | 1M+ | 1 小时视频 ≈ 数十万 token |
| Agent 长轨迹 | 128K-400K | 多步工具调用累积 |
| 极端科研/法律 | 1M-2M | 整本判决书、整篇论文 |

## 相关概念

- [[Tokenization]] — 上下文窗口以 token 为单位
- [[Transformer 架构]] / [[Attention 机制]] — O(n²) 复杂度的根源
- [[../Agent 能力/记忆系统|记忆系统]] — 超过上下文窗口用外部记忆
- [[../Agent 能力/规划与推理|规划与推理]] — 长推理需要长上下文
- [[../../02-Models/Anthropic/Claude Opus 4.8|Claude Opus 4.8]] — 长上下文代表（200K）
- [[../../02-Models/Google/Gemini 3.1 Pro|Gemini 3.1 Pro]] — 超长上下文代表（2M）

## 参考资料

- Liu et al. "Lost in the Middle: How Language Models Use Long Contexts" (2023)
- Press et al. "ALiBi: Train Short, Test Long" (2022)
- Peng et al. "YaRN: Efficient Context Window Extension of LLMs" (2023)
- Tri Dao "FlashAttention" (2022-2024)
- Kwon et al. "Efficient Memory Management for LLM Serving with PagedAttention" (vLLM, 2023)
- Anthropic "Effective context engineering for AI agents" (2024)
