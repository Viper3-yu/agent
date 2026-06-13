---
title: Tokenization
aliases: [分词, Tokenizer]
tags:
  - concept
  - topic/llm
  - status/reviewed
created: 2026-06-13
updated: 2026-06-13
---

# Tokenization（分词）

> 把原始文本切成模型能理解的最小单元——token。这是 LLM 处理文本的第一步，决定了上下文窗口的实际容量和模型的多语言能力。

## 核心思想

神经网络只能处理数字。所以文本必须先转成 token ID 序列：

```
"Hello, world!"
   ↓ Tokenizer
[101, 7592, 1010, 2088, 999]
   ↓ Embedding
[[0.1, 0.3, ...], [...], ...]
```

**为什么不直接用字符？**
- 字符级 vocab 小（~100），但序列太长（100 字 ≈ 100 token），上下文窗口浪费
- 词级 vocab 大（>10 万），但 OOV（未登录词）严重、形态学信息丢失

**折中方案**：subword（子词），把罕见词拆成常见片段。

## 工作原理

### 1. BPE（Byte-Pair Encoding）

GPT 系列使用。核心思路：**迭代合并最高频的字符对**。

```
训练步骤：
1. 初始化 vocab = 所有单字节（256）
2. 重复：找频率最高的相邻对 (A, B)，合并为新符号 AB，加入 vocab
3. 直到 vocab size 达到目标

举例（"low lower lowest"）：
  l o w     → l o w _        频率: 3
  l o w e r → l o w e r _    频率: 1
  合并 "l o" → "lo"：
  lo w _    lo w e r _
```

**变体**：BBPE（Byte-level BPE）以字节为初始 vocab，天然支持任意 Unicode。GPT-2 之后的主流。

### 2. WordPiece

BERT 使用。思路类似 BPE，但合并准则不同：

```
BPE:    选频率最高的对
WordPiece: 选能最大化训练数据似然的合并
        score(A,B) = freq(AB) / (freq(A) × freq(B))
```

倾向于合并"语义相关"的对，而非纯高频对。

### 3. Unigram

XLNet / ALBERT 使用。从大 vocab 开始，**反向剪枝**：

```
1. 初始化大 vocab（如 256k）
2. 计算每个 token 的边际贡献
3. 移除贡献最小的 10-20% token
4. 重复，直到 vocab 达到目标
```

优点：给定多种切分时，能给出**概率最优**的切分（适合需要多输出的场景）。

### 4. SentencePiece

Google 开源的语言无关框架。**把文本当作字节流**（不需要预分词）：

```
输入: "你好世界"
   ↓ 转为字节流: b'\xe4\xbd\xa0\xe5\xa5\xbd\xe4\xb8\x96\xe7\x95\x8c'
   ↓ BPE 或 Unigram 训练
   ↓ 输出切分: ['▁你好', '世界']
```

`_`（U+2581）表示词首空格，天然处理中英混排。Llama / Mistral / Qwen 都用 SentencePiece。

## 各家 LLM 的选型

| 模型 | Tokenizer | 特点 |
|------|-----------|------|
| GPT-4 / GPT-5 | cl100k_base（BBPE） | vocab ~100k，多语言偏弱 |
| Claude 3/4 | 内部 BPE | 多语言平衡 |
| Llama 2/3 | SentencePiece (BPE) | vocab 32k，多语言 OK |
| Mistral | SentencePiece (BPE) | vocab 32k |
| Qwen | SentencePiece (BPE) | vocab 152k，**中文友好** |
| DeepSeek | BBPE | vocab 100k |
| Gemini | SentencePiece (Unigram) | vocab 256k |

## 特殊 Token

每种 tokenizer 都有一组 reserved token：

| Token | 用途 |
|-------|------|
| `<BOS>` / `<s>` | 序列开始 |
| `<EOS>` / `</s>` | 序列结束 |
| `<PAD>` | 批次对齐 |
| `<UNK>` | 未知词 |
| `<SEP>` | 段落分隔 |
| `<MASK>` | 掩码语言模型用 |
| `<|im_start|>` `<|im_end|>` | ChatML 格式（Qwen） |
| `<|tool_call|>` | 工具调用（Llama 3） |

## 关键超参

### Vocab Size 权衡

| Vocab 大小 | 优点 | 缺点 |
|-----------|------|------|
| 小（8k-16k） | 训练快、推理快 | 序列长、压缩比低 |
| 中（32k-64k） | 平衡 | Llama 系列默认 |
| 大（128k-256k） | 压缩比高、序列短 | embedding 表大、稀有 token 训练不足 |

**经验值**：每 token 约 4 字符（英文），1.5 字符（中文）。

### 多语言扩展

加入中文/日文时：
- vocab 增加 20-50k 中文 token
- 中文压缩比从 ~1.5 字符/token 提升到 ~0.5 字符/token
- 但需要重新训练 embedding 行（冷启动）

## 与上下文窗口、计费的关系

**上下文窗口按 token 计**：
- Claude Opus 4.6: 200K tokens ≈ 15 万英文单词 ≈ 500 页书
- GPT-5.5: 400K tokens
- Gemini 3.5 Flash: 2M tokens

**API 计费按 token 计**：
```
计费 token 数 = 输入 tokens + 输出 tokens
中文"你好世界"  → 2 tokens （Qwen）
英文"Hello world" → 2 tokens （Llama）
```

## 中文分词的挑战

中文没有空格，传统 jieba 等基于词典的分词在 LLM 时代**不适用**。LLM tokenizer 用：

1. **BBPE 字节级**：纯字节切分，永远不会 OOV，但效率低
2. **大 vocab 中文子词**：Qwen 152k vocab，中文压缩比高
3. **SentencePiece `_` 词首标记**：天然支持中英混排

**坑点**：
- 同一个中文字符在不同 tokenizer 下 token 数差异可达 3-5 倍
- 这直接导致**API 成本差异**——选对 tokenizer 的模型对中文友好

## 优势与局限

### Tokenization 的优势
- 把开放词汇映射到固定 vocab
- 多语言统一处理
- 高频词用 1 token、低频词拆成多 token，频率自适应
- 支持特殊控制（特殊 token）

### Tokenization 的局限
- **信息丢失**：大小写、标点、whitespace 处理取决于实现
- **数字处理弱**："1234567" 可能拆成多个 token，模型做算术困难
- **不可逆**：emoji、罕见字符可能被拆成单字节，丧失语义
- **词表固化**：新增词（如新品牌名）必须重新训练 tokenizer

### Tokenization-free 探索

为绕过上述问题，近年出现了**无 tokenization**模型：
- **MegaByte / BYT5**：直接处理字节
- **CANINE**：字符级编码
- **BLT**（Byte Latent Transformer，2024 Meta）：动态字节块长度

这些模型在小规模实验中表现有竞争力，但训练效率仍逊于传统 tokenizer 模型。

## 相关概念

- [[Transformer 架构]]——tokenizer 是 Transformer 输入的第一步
- [[Attention 机制]]——计算 token 间的相关性
- [[Token 与上下文窗口]]——token 数决定上下文容量
- [[Embedding]]——token ID → 向量表示
- [[对齐与安全]]——token 级过滤

## 参考资料

- [BPE 原始论文](https://arxiv.org/abs/1508.07909)（Sennrich et al., 2016）
- [SentencePiece 论文](https://arxiv.org/abs/1808.06226)（Kudo & Richardson, 2018）
- [Hugging Face Tokenizers 库](https://github.com/huggingface/tokenizers)
- [tiktoken（OpenAI tokenizer）](https://github.com/openai/tiktoken)
- [Andrej Karpathy: Let's build the GPT Tokenizer](https://youtu.be/zduSFxRajkE)