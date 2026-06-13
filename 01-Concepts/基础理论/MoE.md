---
title: MoE（Mixture of Experts，混合专家）
aliases: [MoE, Mixture of Experts, 混合专家]
tags:
  - concept
  - topic/llm
  - status/reviewed
created: 2026-06-13
updated: 2026-06-13
---

# MoE（Mixture of Experts，混合专家）

> 把一个超大模型的 FFN 层拆成多个"专家"，每个 token 只激活其中少数几个。在**总参数暴涨**的同时保持**推理计算量接近小模型**——这是 2024 年以来 LLM 扩展的标配范式。

## 核心思想

**Dense 模型**：每个 token 走完所有参数。

```
Input x → FFN(x) → Output
           ↑
        所有参数都被激活
```

**MoE 模型**：每个 token 只走 Top-K 个专家。

```
Input x → Router(x) → 选择 Top-K 专家 → 加权求和 → Output
                          ↑
                  只激活少量参数
```

### 关键数字

以 Mixtral 8x7B 为例：

| 指标 | 数值 |
|------|------|
| 总参数 | 46.7B |
| 激活参数（每 token） | ~12.9B |
| 专家数 | 8 |
| 每个 token 激活 | 2（Top-2） |
| 激活比例 | 12.9 / 46.7 ≈ 27.6% |

**收益**：训练时学到了 46.7B 的知识，推理时只花 12.9B 的算力。

## 工作原理

### 1. 专家（Expert）

每个专家就是一个普通的 FFN（前馈网络）：

```
Expert_i(x) = W₂ · σ(W₁ · x + b₁) + b₂
```

不同专家学到了**不同的子任务能力**：
- 某个专家擅长数学符号
- 某个专家擅长代码语法
- 某个专家擅长中文俚语
- ……

### 2. 路由器（Router）

路由器是个小型线性层 + softmax：

```
scores = softmax(W_g · x)        # shape: [num_experts]
top_k_values, top_k_indices = topk(scores, k)
output = Σ (top_k_values[i] · Expert[top_k_indices[i]](x))
```

`W_g` 是可学习参数，shape `[d_model, num_experts]`。

### 3. 训练目标

总损失 = 任务损失 + 负载均衡损失

```
L_total = L_task + α · L_balance

L_balance = num_experts · Σ (fraction_i · mean_score_i)
```

`L_balance` 惩罚"专家不均衡"——防止所有 token 都涌向少数热门专家。

### 4. Capacity Factor（容量因子）

每个专家**最多处理** `C = capacity_factor × (tokens / num_experts)` 个 token。

- `C = 1.0`：理想均匀分配
- `C = 1.25`：允许 25% 溢出
- `C < 1`：强制稀疏，部分 token 被丢弃（token dropping）

## 训练技巧

### 负载均衡

问题：路由器容易陷入"赢家通吃"——某个专家被压垮，其他专家闲置。

解决方案：
1. **辅助损失**（auxiliary loss）：惩罚不均
2. **路由器 z-loss**：防止路由器 logits 过大
3. **专家容量限制**：硬性 drop 超额 token

### 路由器类型

| 类型 | 特点 |
|------|------|
| Top-K 路由（经典） | 简单高效，但不可微 |
| 专家选择路由 | 让专家选 token（DeepSeek-V3 用） |
| 软路由（Hash） | 全可微，训练稳定但稀疏性差 |
| 基址路由（Base Layer） | 简化路由器设计 |

DeepSeek-V3 用的"专家选择路由"很巧妙：
- 每个专家选 Top-C 个 token
- 天然均衡，无须辅助损失
- 但需要限制每个 token 选多少专家

### 训练稳定性

MoE 训练有几个著名陷阱：
- **梯度爆炸**：路由器 logits 易发散 → 用 z-loss
- **专家坍缩**：多个专家学到相同功能 → 负载均衡损失
- **数值不稳定**：FP16 训练溢出 → 用 BF16 + FP32 master

## 推理优化

### 1. 专家并行（Expert Parallelism）

```
GPU 0: 专家 1, 2
GPU 1: 专家 3, 4
GPU 2: 专家 5, 6
GPU 3: 专家 7, 8
```

每个 GPU 只放部分专家，token 通过 All-to-All 通信路由到对应 GPU。

### 2. 预取与缓存

路由器输出有**强局部性**——同一段文本倾向于激活同一组专家。可预取专家权重到 GPU cache。

### 3. 批处理优化

不同 token 激活不同专家，天然支持动态 batching。但要按专家分组计算（grouped GEMM）。

### 4. 推理框架

- **vLLM**：原生 MoE 支持
- **SGLang**：专家并行优化
- **DeepSpeed-MoE**：微软 MoE 推理框架
- **Tutel**：微软自适应并行 MoE 库

## 代表模型

| 模型 | 总参数 | 激活参数 | 专家数 | Top-K | 备注 |
|------|--------|---------|--------|-------|------|
| Mixtral 8x7B | 46.7B | 12.9B | 8 | 2 | 首个开源 MoE 旗舰 |
| Mixtral 8x22B | 141B | 39B | 8 | 2 | 长上下文优化 |
| DeepSeek-V2 | 236B | 21B | 160 | 6 | 细粒度专家 |
| **DeepSeek-V3** | 671B | 37B | 256 | 8 | 专家选择路由 |
| Qwen MoE-A2.7B | 14.3B | 2.7B | 60 | 4 | 阿里小 MoE |
| Qwen MoE-A14B | 75B | 14B | 60 | 4 | 阿里中 MoE |
| Grok-1 | 314B | 86B | 8 | 2 | xAI |
| **MAI-Thinking-1** | 1T | 35B | — | — | 微软，MoE 旗舰 |
| Llama 4 Behemoth | 2T | 288B | 16 | — | Meta 下一代 |

## 与 Dense 模型对比

| 维度 | Dense | MoE |
|------|-------|-----|
| 训练成本 | 高（每 token 算所有参数） | 中（每 token 算 K/N 参数） |
| 推理成本 | 高 | 低 |
| 模型容量 | 固定 | 总参数可很大 |
| 显存占用 | 等于参数数 | 仍需加载所有专家 |
| 部署复杂度 | 低 | 高（专家并行） |
| 微调难度 | 低 | 中（部分专家可冻结） |
| 知识冲突 | 严重 | 分散到专家，较轻 |

## 优势与局限

### 优势
- **性价比高**：用推理时间换参数容量
- **可扩展**：加专家 = 加容量，无需从头训练
- **天然多任务**：不同专家学不同能力
- **稀疏性**：理论上极大

### 局限
- **显存压力大**：必须加载所有专家（即使不激活）
- **通信开销**：专家并行需要高速互联（NVLink / InfiniBand）
- **调参复杂**：路由器、容量因子、均衡损失超参多
- **推理延迟不稳定**：专家调度不均时延迟抖动
- **微调灾难性遗忘**：微调时部分专家可能完全失活

## 相关概念

- [[Transformer 架构]]——MoE 替换的是 FFN 层
- [[Attention 机制]]——MoE 与 Attention 是 LLM 的两大核心
- [[量化]]——MoE 量化更复杂（专家稀疏激活难估计范围）
- [[RAG 检索增强生成]]——RAG 减少 MoE 的容量压力

## 参考资料

- [Mixtral 论文](https://arxiv.org/abs/2401.04088)
- [DeepSeek-V2 / V3 论文](https://arxiv.org/abs/2405.04434)
- [GShard: Scaling Giant Models with Conditional Computation](https://arxiv.org/abs/2006.16668)
- [Switch Transformer](https://arxiv.org/abs/2101.03961)
- [MegaBlocks: Efficient Sparse Training with Mixture-of-Experts](https://arxiv.org/abs/2211.15841)