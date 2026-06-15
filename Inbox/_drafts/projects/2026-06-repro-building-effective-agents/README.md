---
title: 复现 Anthropic《Building effective agents》
aliases: [Building effective agents 复现, effective-agents-repro]
tags:
  - project
  - topic/agent
  - status/draft
created: 2026-06-15
updated: 2026-06-15
status: 进行中
---

# 复现 Anthropic《Building effective agents》

> 用 LangGraph 把 Anthropic 2024 年那篇《Building effective agents》提到的 5 个原始工作流范式**一个一个跑通**，每个范式都留可运行的最小 demo + 评测记录。

## 项目目标

- **核心目标**：在 LangGraph 中复现 Prompt Chaining / Routing / Parallelization / Orchestrator-Workers / Evaluator-Optimizer 5 个范式
- **次要目标**：横向对比 5 个范式在不同任务（写代码 / 写文章 / 检索 + 综合）上的成本/质量/延迟
- **远期目标**：提炼出"什么任务该用哪个范式"的决策树，写成 vault 里的 `01-Concepts/Agent 架构/范式选择决策树`

## 背景与动机

读了 [[../../Clippings/llm-wiki|LLM Wiki]] 之后意识到：**对 Agent 范式的理解，光看论文是假的，跑过才是真的**。vault 里 04-Frameworks/ 框架笔记多但 05-Projects/ 全空，正好缺这个空白。

Anthropic 那篇 5 个范式是 "workflow vs agent" 讨论的起点，先把这 5 个打牢再去看 AutoGen / CrewAI 之类的"高阶框架"。

## 技术方案

| 范式 | 最小 demo | 输入 | 期望输出 |
|------|----------|------|---------|
| **Prompt Chaining** | 写营销文案：选题→大纲→初稿→质检 | 主题 + 受众 | 3 段式营销文案 |
| **Routing** | 客服问题分类→不同处理流 | 用户问题 | 分类标签 + 对应回复 |
| **Parallelization** | 论文摘要：3 路并行（背景/方法/结论）→ 合并 | arXiv PDF | 200 字综述 |
| **Orchestrator-Workers** | 编程任务：主 LLM 拆分子任务→并发调 coding LLM | 模糊需求 | 完整代码 + 测试 |
| **Evaluator-Optimizer** | 翻译：写译文→评估→改进→循环 | 中文段落 | 润色过的英文 |

**栈选型**：
- **LangGraph**（[[../../../04-Frameworks/LangGraph/LangGraph|LangGraph]]）— 状态图最贴近 5 个范式
- **GPT-4o-mini** 主力 / **Claude Sonnet 4.6** 备份 — 便宜 + 够用
- **LangSmith** 做 trace，方便定位哪个节点出问题
- 每个范式一个 `demos/<name>/` 子目录，含 `graph.py` + `test.py` + `eval.csv`

## 里程碑

- [ ] **M1（2 周）** — Prompt Chaining 跑通 + 评测 baseline
- [ ] **M2（4 周）** — Routing + Parallelization 跑通
- [ ] **M3（6 周）** — Orchestrator-Workers 跑通（最复杂）
- [ ] **M4（8 周）** — Evaluator-Optimizer 跑通
- [ ] **M5（10 周）** — 横向对比表 + 决策树笔记
- [ ] **M6（12 周）** — 在 vault 里发 5 篇复盘笔记，每篇含 trace 截图 + 踩坑日志

## 已读资料

- [Anthropic《Building effective agents》](https://www.anthropic.com/research/building-effective-agents) — 范式定义原文
- [LangGraph 官方 Quickstart](https://langchain-ai.github.io/langgraph/) — 跑通第一个图
- [[../../../04-Frameworks/LangGraph/LangGraph|LangGraph 笔记]] — 框架基础
- [[../../../01-Concepts/Agent 架构/ReAct 架构|ReAct 架构]] — ReAct vs 5 个 workflow 的关系

## 待读资料

- Lilian Weng "LLM Agent" 综述（铺垫）
- Anthropic "How we built our multi-agent research system"（Orchestrator-Workers 的真实案例）

## 进展日志

### 2026-06-15
- 项目立项：从方案 §5.1 采纳，进入 Active
- 创建项目文件夹骨架
- 选定 LangGraph + GPT-4o-mini 栈
- 待办：先把 LangGraph 环境搭起来

## 风险与回避

| 风险 | 应对 |
|------|------|
| LangGraph API 大改 | 锁版本（`langgraph==0.2.x`） |
| 评测主观难量化 | 用 LLM-as-judge + 5 分制 |
| 跑到一半变 5 个玩具 | 严格要求每个范式跑真实任务 + 记 trace |
| 拖延 | M1 完成后立即发 vault 复盘，给外部约束 |

## 产出物（最终交付）

1. **5 个跑通的 demo**（每范式 1 个）
2. **5 篇 vault 复盘笔记**（每范式 1 篇，含 trace + 踩坑 + 反思）
3. **1 张决策树笔记**（什么任务选什么范式）
4. **1 篇博客**（对外分享，可选）

## 相关资源

- → [[../../../00-MOC/项目看板|项目看板 MOC]]
- → [[../../../00-MOC/Agent 全景地图|Agent 全景地图]]
- → [[../../../00-MOC/框架与工具地图|框架与工具地图]]
- → [[../README|Active 看板]]
- → [[../../../Templates/tpl-项目 README|项目 README 模板]]
