---
title: LoRA 与 QLoRA
aliases: [LoRA, QLoRA, Low-Rank Adaptation]
tags:
  - concept
  - status/seed
category: 参数高效微调
base_models: [LLaMA, Mistral, Qwen, ChatGLM]
hardware: QLoRA 最低 16GB 显存；LoRA 需 24GB+
created: 2026-06-12
updated: 2026-06-12
---

# LoRA 与 QLoRA

## 概述

LoRA（Low-Rank Adaptation）和 QLoRA 是当前最流行的参数高效微调（PEFT）方法。LoRA 通过低秩矩阵分解，只训练极少量参数（约占原模型的 0.1%-1%），大幅降低显存和计算需求。QLoRA 在此基础上引入 4-bit 量化，使得消费级 GPU 也能微调大模型。

## 核心原理

### LoRA

LoRA 的核心思想是：预训练模型的权重更新矩阵 ΔW 具有低秩特性，可以用两个小矩阵的乘积来近似。

```
原始前向传播:  h = Wx
LoRA 前向传播:  h = Wx + (B·A)x

其中:
  W ∈ R^(d×d)  — 原始权重（冻结）
  A ∈ R^(r×d)  — 降维矩阵（可训练）
  B ∈ R^(d×r)  — 升维矩阵（可训练）
  r << d       — 秩（通常 8-64）

可训练参数量: 2 × d × r（远小于 d²）
缩放因子:     ΔW = (α/r) × B·A
```

### QLoRA

QLoRA 在 LoRA 基础上增加三个关键技术：

1. **4-bit NormalFloat (NF4)**：对预训练权重进行 4-bit 量化，信息论最优的量化格式
2. **双重量化（Double Quantization）**：对量化常数再次量化，进一步节省显存
3. **分页优化器（Paged Optimizer）**：GPU 显存不足时自动将优化器状态卸载到 CPU

```
QLoRA 显存计算:
  7B 模型:  4-bit 权重 ~3.5GB + LoRA 参数 ~0.1GB + 优化器 ~1GB ≈ 5GB
  65B 模型: 4-bit 权重 ~33GB + LoRA 参数 ~0.5GB + 优化器 ~5GB ≈ 40GB
```

## 前置条件

### 数据要求

| 项目 | 要求 |
|------|------|
| 数据量 | 100-10,000 条高质量样本；通常 1,000 条即可获得不错效果 |
| 数据格式 | Alpaca 格式（instruction-input-output）或 conversation 格式 |
| 数据质量 | 高质量 > 高数量；避免重复、矛盾、低质量样本 |

### 硬件需求

| 项目 | 最低配置 | 推荐配置 |
|------|----------|----------|
| GPU | RTX 3090 24GB（QLoRA 7B） | A100 80GB（LoRA 70B） |
| 显存 | 16GB（QLoRA 7B 模型） | 48GB+（LoRA 全精度） |
| 内存 | 32GB | 64GB+ |

## 训练流程

### 步骤一：数据准备

```python
from datasets import load_dataset

# 加载数据集
dataset = load_dataset("json", data_files="train.json")

# 格式化为 ChatML 模板
def format_chat(example):
    messages = [
        {"role": "system", "content": "你是一个专业的助手。"},
        {"role": "user", "content": example["instruction"] + example.get("input", "")},
        {"role": "assistant", "content": example["output"]}
    ]
    return {"messages": messages}

dataset = dataset.map(format_chat)
```

### 步骤二：配置训练参数

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model

# QLoRA 量化配置
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,
)

# 加载量化模型
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=bnb_config,
    device_map="auto",
)

# LoRA 配置
lora_config = LoraConfig(
    r=16,                        # 秩
    lora_alpha=32,               # 缩放因子（通常 = 2*r）
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM",
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# 输出示例: trainable params: 4,194,304 || all params: 6,742,609,920 || trainable%: 0.0622
```

### 步骤三：启动训练

```bash
# 使用 Hugging Face PEFT + TRL
python train.py \
    --model_name meta-llama/Llama-2-7b-hf \
    --dataset_path ./data/train.json \
    --output_dir ./output \
    --num_epochs 3 \
    --batch_size 4 \
    --learning_rate 2e-4

# 使用 Unsloth（2x 加速）
python train_unsloth.py

# 使用 Axolotl
axolotl train lora_config.yml
```

### 步骤四：评估与部署

```python
from peft import PeftModel
from transformers import AutoModelForCausalLM

# 加载基础模型 + LoRA 权重
base_model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b-hf")
model = PeftModel.from_pretrained(base_model, "./output/checkpoint-final")

# 合并 LoRA 权重到基础模型（可选，推理时无需 PEFT）
merged_model = model.merge_and_unload()
merged_model.save_pretrained("./merged_model")

# 导出为 GGUF（用于 llama.cpp / Ollama）
# 使用 llama.cpp 的 convert 脚本
```

## 关键超参数

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| r (rank) | 8-64 | 低秩矩阵的秩；越大效果越好但显存越高 |
| lora_alpha | 通常 = 2*r | 缩放因子，控制 LoRA 更新的权重 |
| target_modules | q_proj, v_proj | 应用 LoRA 的层；扩展到 k_proj, o_proj 可提升效果 |
| lora_dropout | 0.05-0.1 | Dropout 比例，防过拟合 |
| learning_rate | 1e-4 ~ 3e-4 | LoRA 学习率，高于全量微调 |
| num_epochs | 1-3 | 数据量少时增加 epoch |
| batch_size | 4-8 | 根据显存调整 |
| warmup_ratio | 0.03-0.1 | 学习率预热比例 |

## 与其他方法对比

| 特性 | LoRA | QLoRA | 全量微调 | Adapter |
|------|------|-------|----------|---------|
| 可训练参数 | 0.1%-1% | 0.1%-1% | 100% | 1%-5% |
| 显存占用 | 中 | 低 | 高 | 中 |
| 训练速度 | 快 | 快（略慢于 LoRA） | 慢 | 中 |
| 效果 | 接近全量微调 | 接近 LoRA | 最好 | 接近 LoRA |
| 推理开销 | 合并后无额外开销 | 合并后无额外开销 | 无额外开销 | 有额外延迟 |
| 适用场景 | 通用 | 显存受限 | 充足算力 | 旧架构兼容 |

## 常见问题与排坑

- **显存不足**：降低 batch_size、启用 gradient checkpointing、使用 QLoRA
- **效果不佳**：尝试增大 rank r、扩展 target_modules、增加数据量
- **过拟合**：增加 lora_dropout、减少 epoch、使用更多数据
- **训练不稳定**：降低 learning_rate、增加 warmup_steps
- **合并权重后质量下降**：确保 merge_and_unload() 在相同精度下执行
- **target_modules 选择**：不同模型架构的层名不同，需查看模型结构确认

## 参考资料

- [[微调总览]]
- [LoRA 论文](https://arxiv.org/abs/2106.09685)
- [QLoRA 论文](https://arxiv.org/abs/2305.14314)
- [Hugging Face PEFT](https://github.com/huggingface/peft)
- [Unsloth](https://github.com/unslothai/unsloth)
