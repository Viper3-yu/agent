---
title: Harness 总览
status: seed
tags:
  - agent
  - harness
  - evaluation
  - testing
category: framework
url: ""
created: 2026-06-10
updated: 2026-06-10
related:
  - "[[../Agent 可观测性/评估框架总览]]"
  - "[[../Agent 可观测性/可观测性总览]]"
  - "[[../Agent 安全/Agent 安全总览]]"
---

# Harness 总览

## 概述

Agent Harness Engineering 是一套系统化的基础设施，用于驱动（drive）、观察（observe）和评判（judge）Agent 行为。它填补了从开发到生产之间的"缺失中间地带"，是将 Agent 从"能跑"推向"可信赖"的关键工程实践。

Harness 是一个**主动式（proactive）**的评估管道：你设计测试用例，注入到 Agent 中，收集输出并打分。它与可观测性（Observability）形成互补——可观测性是被动地观察生产中发生了什么，Harness 是主动地构造场景来验证 Agent 是否符合预期。

| 维度 | Harness | Observability |
|------|---------|---------------|
| 驱动方式 | 主动：你设计测试用例 | 被动：你观察真实流量 |
| 环境 | 开发/预发布/staging | 生产 |
| 目标 | 验证功能、安全、性能 | 发现异常、定位故障 |
| 时机 | 发布前 | 发布后 |
| 典型产出 | 评估报告、回归基线 | 告警、Trace、Dashboard |

## 评估维度

| 维度 | 指标 | 计算方式 | 参考值 |
|------|------|----------|--------|
| 任务完成度 | Task Completion Rate | 成功完成任务 / 总任务数 | > 90% |
| 幻觉率 | Hallucination Rate | 事实错误输出数 / 总输出数 | < 5% |
| 响应延迟 | Latency (P95) | 95 分位响应时间 | 依场景而定 |
| 单次成本 | Cost per Query | 单次查询 token/API 成本 | 依预算而定 |
| 安全违规率 | Safety Violation Rate | 触发安全规则数 / 总请求数 | 0% |
| 用户满意度 | User Satisfaction | 用户主观评分（1-5） | > 4.0 |

### 四类 Harness

#### 1. Eval Harness（评估线束）

核心管道：`Input → Agent → Output → Scoring`

最常见的 Harness 类型。给定一组测试输入，运行 Agent，对输出进行自动化打分。评分维度包括准确性、相关性、忠实度（faithfulness）、格式合规等。

适用场景：RAG 质量评估、指令遵循测试、工具调用正确性验证。

#### 2. Test Harness（测试线束）

聚焦于**可靠性**：回归测试（改动后行为是否退化）、边界测试（极端输入下的表现）、对抗性测试（adversarial inputs）。

与传统单元测试的区别：Agent 的"断言"往往不是精确匹配，而是语义相似度或 LLM-as-Judge。

#### 3. Conversation Harness（对话线束）

模拟**多轮用户交互**，验证 Agent 在上下文累积、意图切换、澄清追问等场景下的端到端表现。

典型做法：用另一个 LLM 扮演用户，按预设脚本与目标 Agent 对话，最终由 Judge 评估整体对话质量。

#### 4. Red-team Harness（红队线束）

专注于**安全攻击模拟**：Prompt Injection、Jailbreak、数据外泄（data exfiltration）、权限提升。

红队 Harness 通常由安全团队驱动，产出是漏洞清单和防御建议。与 [[../Agent 安全/Agent 安全总览|Agent 安全总览]] 紧密关联。

### 核心组件

一个完整的 Harness 系统包含以下组件：

1. **Test Scenarios（测试场景）**：定义输入、预期行为、上下文条件
2. **Evaluation Metrics（评估指标）**：Task Completion Rate、Hallucination Rate、Safety Violation Rate 等
3. **Scoring Functions（评分函数）**：精确匹配、语义相似度、LLM-as-Judge、正则规则
4. **Report Generation（报告生成）**：结构化评估报告，支持历史对比
5. **CI Integration（CI 集成）**：将评估管道嵌入 CI/CD，阻断不合格的发布

### Harness Engineering 生命周期

```
Design Scenarios → Build Harness → Run Evaluation → Analyze Results → Iterate
     ↑                                                                    |
     └────────────────────────────────────────────────────────────────────┘
```

1. **Design Scenarios**：根据用户故事、历史 bug、安全威胁建模设计测试场景
2. **Build Harness**：搭建管道，连接 Agent、数据集、评分器
3. **Run Evaluation**：批量执行，收集输出和指标
4. **Analyze Results**：识别薄弱环节，定位失败模式
5. **Iterate**：优化 Agent 或扩充测试场景，回到步骤 1

## 使用方法

### 输入格式

```json
{
  "harness_type": "eval",
  "dataset": "test_set.jsonl",
  "agent_config": {
    "model": "gpt-4o",
    "tools": ["search", "calculator"],
    "system_prompt": "你是客服助手..."
  },
  "metrics": ["task_completion", "hallucination", "safety"],
  "thresholds": {
    "task_completion": 0.90,
    "hallucination": 0.05,
    "safety": 0.00
  }
}
```

### 输出格式

```json
{
  "run_id": "run-20260612-001",
  "total_cases": 200,
  "pass_rate": 0.93,
  "per_metric": {
    "task_completion": { "score": 0.95, "pass": true },
    "hallucination": { "score": 0.03, "pass": true },
    "safety": { "score": 0.00, "pass": true }
  },
  "regression_vs_baseline": { "delta": -0.02, "significant": false },
  "failed_cases": ["tc-012", "tc-089"]
}
```

### 代码示例

```yaml
# 示例：将 Eval Harness 嵌入 CI Pipeline
eval-gate:
  stage: test
  script:
    - python run_eval.py --dataset test_set.jsonl --threshold 0.85
  rules:
    - if: $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"
  allow_failure: false  # 阻断不合格发布
```

### LLM-as-Judge 的可靠性工程

使用 LLM 作为评分器时需要注意：
- **Position Bias**：LLM 倾向于偏好第一个（或最后一个）选项
- **Verbosity Bias**：LLM 倾向于偏好更长的回答
- **自我偏好**：GPT-4 倾向于给 GPT-4 的输出更高分

缓解策略：随机化输出顺序、使用多个 Judge 取共识、用人工标注校准 Judge 准确率。

## 适用场景

- RAG 系统的端到端质量评估
- Agent 工具调用正确性和安全性验证
- Prompt 迭代效果的量化对比
- CI/CD 流水线中的质量门禁
- 与 [[../../01-Concepts/Agent 构建模式/评估者-优化者模式 Evaluator-Optimizer|Evaluator-Optimizer 模式]] 的工程化集成

## 局限性

- 测试场景的构建和维护需要持续投入
- LLM-as-Judge 存在偏好偏差，需要定期校准
- 评估成本随测试规模线性增长（LLM 调用费用）
- 非确定性输出导致测试结果存在方差，需多次运行取统计值

## 与其他评估方法的对比

| 特性 | Eval Harness | RAGAS | promptfoo | DeepEval | LangSmith |
|------|-------------|-------|-----------|----------|-----------|
| 定位 | 通用 Agent 评估 | RAG 评估专用 | 通用 LLM 评估 | 端到端评估框架 | 可观测性 + 评估 |
| RAG 评估 | ★★★★ | ★★★★★ | ★★★ | ★★★★ | ★★★ |
| Agent 评估 | ★★★★★ | ★★ | ★★★ | ★★★★ | ★★★ |
| 多轮对话 | ★★★★ | ★★ | ★★ | ★★★ | ★★★ |
| 安全评估 | ★★★ | ★ | ★★★ | ★★★ | ★★ |
| CI 集成 | ★★★★ | ★★★ | ★★★★★ | ★★★★ | ★★★★ |
| 自定义指标 | ★★★★ | ★★★ | ★★★★★ | ★★★★ | ★★★ |
| 开源/付费 | 开源 | 开源 | 开源 | 开源+付费 | 付费为主 |

## 参考资料

- [[../Agent 可观测性/评估框架总览]] - 评估框架的更广泛视角
- [[../Agent 可观测性/可观测性总览]] - Harness 的被动式互补
- [[../Agent 安全/Agent 安全总览]] - Red-team Harness 的安全上下文
- [[../../01-Concepts/Agent 构建模式/评估者-优化者模式 Evaluator-Optimizer]] - Harness 的模式抽象
- [[../../06-Resources/微调/微调总览]] - Harness 数据如何流入训练管道
