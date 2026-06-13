---
title: 路由模式 Routing
aliases: [Routing, 分类路由]
tags:
  - concept
  - topic/agent
  - status/seed
category: workflow
complexity: low
source: "Anthropic: Building Effective Agents"
created: 2026-06-12
updated: 2026-06-12
---

# 路由模式 Routing

> 根据输入的类型或特征，将其分发到不同的处理路径。类似编程中的 switch-case。

## 模式定义

Routing 模式通过一个分类步骤判断输入的类型或意图，然后将其分发到对应的专业化处理路径。每条路径可以拥有独立的 Prompt、模型配置甚至不同的处理逻辑，从而实现对不同类型输入的最优处理。分类步骤通常使用小模型或简单规则以降低延迟和成本。

## 适用场景

1. 输入类型多样，需要不同处理逻辑
2. 客服系统（按问题类型路由：退款、技术支持、投诉）
3. 代码审查（按语言路由到不同检查器）
4. 多语言内容处理（按语种路由到对应翻译管道）
5. 请求分类后分发到不同的专业 Agent

## 不适用场景

1. 输入类型单一，无需分类 — 直接处理即可
2. 所有输入需要相同处理逻辑 — 使用 [[Prompt Chaining]] 更简单
3. 分类本身非常复杂且需要多步推理 — 分类器可能成为瓶颈

## 架构图

```
输入 → [分类器/路由器]
          │
          ├─ 类型A → [处理A: 专用Prompt] → 输出A
          │
          ├─ 类型B → [处理B: 专用Prompt] → 输出B
          │
          ├─ 类型C → [处理C: 专用Prompt] → 输出C
          │
          └─ 未知  → [Fallback处理]      → 输出/错误
```

## 核心流程

1. 接收输入，提取用于分类的特征
2. 分类器判断输入类型（可用 LLM、小模型或规则引擎）
3. 根据分类结果选择对应的处理路径
4. 将输入转发到选定路径的专业 Prompt 进行处理
5. 返回处理结果；若分类不确定，走 fallback 路径

## 伪代码示例

```python
def route(input_text):
    # 步骤1: 分类（使用小模型或规则，快速低成本）
    category = classify(input_text)  # "refund" | "tech_support" | "complaint" | "unknown"

    # 步骤2: 路由到专业化处理路径
    handlers = {
        "refund": refund_handler,
        "tech_support": tech_support_handler,
        "complaint": complaint_handler,
    }

    handler = handlers.get(category, fallback_handler)
    return handler(input_text)

def classify(input_text):
    """分类步骤：快速、低成本"""
    return call_small_model(
        prompt=f"将以下客户消息分类为: refund, tech_support, complaint\n\n{input_text}"
    )
```

## 优势

- 每条路径的 Prompt 可高度专业化，提升处理质量
- 分类步骤快速低成本（可用小模型或规则）
- 新增路径只需添加新的处理分支，扩展性好
- 各路径独立，可单独优化和测试

## 局限与风险

- 分类错误会导致整条路径结果错误（错误传播）
- 路径数量增多时，分类器准确率可能下降
- 边界情况（输入跨越多个类型）难以处理

## 关键决策点

| 决策 | 建议 | 理由 |
|------|------|------|
| 分类器选型 | 小模型或规则引擎 | 分类步骤要快速且低成本 |
| Fallback 策略 | 设置默认路径或请求人工 | 处理未识别的输入，避免丢弃 |
| 路径粒度 | 按处理逻辑差异划分 | 差异大则分，差异小则合并 |
| 路径数量 | 控制在 5-10 条以内 | 过多会降低分类准确率 |

## 与其他模式的关系

- 比 [[Prompt Chaining]] 更灵活，适合非线性流程
- 可与 [[编排者-工人模式 Orchestrator-Worker]] 结合：编排者先路由再分配
- 每条路径内部可以是 [[Prompt Chaining]] 链
- 与 [[并行化 Parallelization]] 可组合：对每条路径并行执行子任务

## 参考资料
- [[构建模式总览]]
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
