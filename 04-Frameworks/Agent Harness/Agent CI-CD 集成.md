---
title: Agent CI-CD 集成
status: reviewed
tags:
  - topic/agent
  - harness
  - topic/coding
category: framework
url: ""
created: 2026-06-10
updated: 2026-06-10
related:
  - "[[Harness 总览]]"
  - "[[评测管线设计]]"
  - "[[对抗测试 Red-Teaming]]"
---

# Agent CI/CD 集成

## 概述

传统软件 CI/CD 的核心假设是**确定性**：相同输入产生相同输出，测试结果非 pass 即 fail。Agent 系统打破了这些假设——输出非确定性、评测成本高昂、质量判定是多维度连续值。本文探讨如何将 Agent 评测与测试集成到 CI/CD 流水线中，涵盖非确定性处理、成本管控、部署门禁和生产监控等关键议题。

| 维度 | 传统软件 | Agent 系统 |
|------|---------|-----------|
| 输出确定性 | 完全确定 | 非确定性（同一输入不同响应） |
| 质量判定 | 二元 pass/fail | 模糊评分（多维度连续值） |
| 评测成本 | 近乎零（CPU 计算） | 昂贵（LLM 调用消耗 token 和时间） |
| 评测维度 | 功能正确性为主 | 准确性、安全性、成本、延迟等多维度 |
| 回归检测 | 精确 diff | 统计分布偏移 |

## 评估维度

| 维度 | 指标 | 计算方式 | 参考值 |
|------|------|----------|--------|
| 质量门禁 | eval score 回归幅度 | 当前分数 vs 基线分数 | 不超过基线 -3% |
| 安全门禁 | critical safety violations | 安全扫描违规计数 | 0 个 |
| 成本门禁 | 每次查询平均成本 | 总成本 / 总查询数 | <= $0.02 |
| 延迟门禁 | P95 响应延迟 | 95 分位延迟 | <= 3000ms |

任一门禁失败即阻止部署，并生成诊断报告，指出退化的测试用例和可能原因。

## 使用方法

### 输入格式

```json
{
  "ci_config": {
    "trigger": "pull_request",
    "paths": ["prompts/**", "src/agent/**"],
    "stages": ["lint", "unit", "integration", "e2e", "safety"],
    "sampling_rate": 0.10,
    "use_mock_llm": true
  },
  "deployment_gates": {
    "quality": { "metric": "eval_score", "threshold": -0.03 },
    "safety": { "metric": "critical_violations", "threshold": 0 },
    "cost": { "metric": "avg_cost_per_query", "threshold": 0.02 },
    "latency": { "metric": "p95_latency_ms", "threshold": 3000 }
  }
}
```

### 输出格式

```json
{
  "pipeline_id": "ci-20260612-001",
  "status": "passed",
  "stages": {
    "lint": { "status": "passed", "duration_s": 10, "cost": 0 },
    "unit": { "status": "passed", "duration_s": 30, "cost": 0 },
    "integration": { "status": "passed", "duration_s": 120, "cost": 0.01 },
    "e2e": { "status": "passed", "duration_s": 300, "cost": 0.50 },
    "safety": { "status": "passed", "duration_s": 180, "cost": 0.20 }
  },
  "gate_results": {
    "quality": { "passed": true, "delta": -0.01 },
    "safety": { "passed": true, "violations": 0 },
    "cost": { "passed": true, "avg_cost": 0.015 },
    "latency": { "passed": true, "p95": 2500 }
  }
}
```

### 代码示例

```yaml
# GitHub Actions 示例
name: Agent Eval Pipeline
on:
  pull_request:
    paths: ['prompts/**', 'src/agent/**']
  schedule:
    - cron: '0 3 * * *'  # Nightly deep eval

jobs:
  lint-and-typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint:prompts
      - run: npx tsc --noEmit

  unit-tests:
    needs: lint-and-typecheck
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test:unit -- --coverage

  e2e-eval:
    needs: unit-tests
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx promptfoo eval --config eval/ci.yaml
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      - run: npm run eval:gate -- --threshold 0.80

  nightly-deep-eval:
    needs: unit-tests
    if: github.event_name == 'schedule'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx promptfoo eval --config eval/full.yaml
      - run: npm run eval:safety -- --suite red-team-full
```

### CI Pipeline 六阶段设计

```
PR / Push 触发
    │
    ▼
┌─────────────────┐
│ Stage 1: Lint   │  prompt 模板语法检查、config 类型校验
│ & Type-check    │  ⏱ ~10s, 💰 $0
└────────┬────────┘
         ▼
┌─────────────────┐
│ Stage 2: Unit   │  工具函数、工具代码（确定性部分）
│ Tests           │  ⏱ ~30s, 💰 $0
└────────┬────────┘
         ▼
┌─────────────────┐
│ Stage 3:        │  Agent + Tools + Mock LLM
│ Integration     │  ⏱ ~2min, 💰 ~$0.01
└────────┬────────┘
         ▼
┌─────────────────┐
│ Stage 4: E2E    │  真实 LLM, Golden Dataset, 评分
│ Evaluation      │  ⏱ ~5min, 💰 ~$0.50
└────────┬────────┘
         ▼
┌─────────────────┐
│ Stage 5: Safety │  Red-team 抽样 + 性能基准
│ & Perf Gate     │  ⏱ ~3min, 💰 ~$0.20
└────────┬────────┘
         ▼
    ✅ Merge / ❌ Block
```

**关键设计原则**：快速失败（fail fast）。Stage 1-2 成本为零且秒级完成，拦截大部分低级错误；昂贵的 E2E 评测仅在前序阶段全部通过后执行。

### 处理非确定性

Agent 输出的非确定性是 CI 集成的最大挑战。四种策略组合使用：

**1. 统计测试（Statistical Testing）** — 同一测试用例运行 N 次（N=5~10），检查通过率分布。例如 5 次中 4 次 pass 即 pass_rate=80%，超过 70% 阈值则判定 PASS。

**2. 阈值判定（Threshold-based Pass）** — 在 eval dataset 上计算综合得分，超过阈值即通过。同时与基线版本对比，关注**相对变化**而非绝对值，允许小幅波动但阻止显著退化。

**3. Temperature=0 可复现模式** — CI 环境中将 temperature 设为 0 以提升确定性。注意这不能消除所有随机性（如 top-p 采样、并行工具调用顺序），但能大幅减少方差。

### 成本管控

LLM 调用费用是 Agent CI/CD 的核心约束。采用分层评测策略：

```
CI（每次 PR/Push）: 10% 抽样 + Mock LLM + 烟雾测试  → ~$0.50/次
Nightly（每晚定时）: 100% 全量 + Red-team + 性能基准  → ~$5.00/次
Release（发布前触发）: 全量 + A/B 对比 + 安全审查    → ~$20.00/次
```

**成本控制手段**：
- **LLM Response Caching**：对确定性测试用例缓存响应，避免重复调用
- **Budget Circuit Breaker**：设置预算上限，超支自动中断流水线
- **Sampling Strategy**：CI 阶段仅测试 10% 用例，Nightly 运行 100%

### GitOps for Prompts

将 prompt 和 system template 纳入版本控制，获得与代码相同的管理能力：

- **Prompt Diff in PR**：PR review 时查看 prompt 变更内容，而非隐藏在代码中的字符串拼接
- **A/B Deployment**：通过 feature flag 部署多版本 prompt，按流量比例分配
- **Rollback Strategy**：prompt 变更导致质量退化时，一键回滚到上一个通过评测的版本

```
prompts/
├── v2.1.0/  (system.md, tools.yaml, eval_results.json)
├── v2.0.0/  (system.md, tools.yaml)
└── current -> v2.1.0
```

### 工具选型

| 工具 | 定位 | CI 集成方式 |
|------|------|------------|
| **promptfoo** | 通用 LLM eval 框架 | YAML 配置 + CLI + GitHub Action |
| **DeepEval** | pytest 风格评测 | `@pytest.mark.evaluate` 装饰器 |
| **LangSmith** | 在线评测平台 | SDK 调用 + Dashboard 告警 |

promptfoo 适合快速上手，DeepEval 适合 Python 生态，LangSmith 适合需要持续生产监控的场景。对于定制需求，也可编写自研评分脚本，通过退出码对接 CI。

## 适用场景

- Agent 代码或 Prompt 变更的自动化质量验证
- 模型版本升级前后的回归测试
- 多团队协作时的 Prompt 变更审查
- 生产环境持续质量监控和告警
- 合规要求下的安全审计自动化

## 局限性

- 非确定性输出导致测试结果存在方差，需多次运行取统计值
- LLM 调用成本限制了 CI 阶段的测试覆盖率
- 温度设为 0 不能完全消除随机性
- 评测数据集需要持续维护，否则快照陈旧导致结果失真
- 跨模型、跨版本的评测结果对比需要标准化

## 与其他评估方法的对比

| 特性 | CI/CD 集成 | 手动评测 | 定期基准测试 | 生产监控 |
|------|-----------|----------|------------|----------|
| 触发时机 | 每次 PR/Push | 按需 | 定期（Nightly） | 持续 |
| 自动化程度 | 高 | 低 | 高 | 高 |
| 反馈速度 | 分钟级 | 小时级 | 小时级 | 实时 |
| 成本 | 中（分层控制） | 高 | 中 | 低 |
| 覆盖度 | 可控（抽样） | 全量 | 全量 | 全量 |
| 阻断能力 | 强（门禁） | 弱 | 中 | 弱（告警） |

## 参考资料

- [[Harness 总览]] - Agent Harness 整体架构
- [[评测管线设计]] - 评测流水线详细设计
- [[对抗测试 Red-Teaming]] - 安全评测方法论
- promptfoo 官方文档: https://promptfoo.dev/docs
- DeepEval 文档: https://docs.confident-ai.com
- LangSmith 评测指南: https://docs.smith.langchain.com
