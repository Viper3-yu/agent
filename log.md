---
title: 知识库变更日志
tags:
  - meta/log
  - status/reviewed
created: 2026-06-13
updated: 2026-06-13
---

# log.md

> 知识库演化的时间线。每个操作按 `## [YYYY-MM-DD] <操作类型> | <简短描述>` 格式记录，可用 `grep "^## \[" log.md | tail -10` 快速查看最近 10 条。

**操作类型**：
- `ingest` — 摄入新原始资料到 wiki
- `query` — 回答老大问题（不产生 wiki 变更）
- `lint` — 健康检查（诊断报告）
- `fix` — 修复断链 / frontmatter / 标签
- `refactor` — 结构重组
- `draft` — Claudian 起草（待审）
- `apply` — 应用草稿到正式位置
- `meta` — 元规范更新（KB 构建规范等）

---

## [2026-06-13] meta | 新建 log.md + 重写 KB 构建规范对齐 Karpathy 模式

发现老大之前 clip 过 Karpathy 的 [llm-wiki](Clippings/llm-wiki.md) 文章，整 vault 实际上是按"原始资料 → wiki → schema"三层模式构建的。
补齐 Karpathy 模式要求的两个核心文件：
- 本文件（log.md）— 时间线
- `00-MOC/KB 构建规范.md` — 已重写为符合 Karpathy 描述的"schema"格式

---

## [2026-06-13] meta | KB 构建规范 v2 落地

把第一版（凭直觉）改成 v2 显式对齐 Karpathy "LLM Wiki" 三层架构：
- 第一节变成"三层架构映射"表（Raw / Wiki / Schema 各自对应 vault 的哪几块）
- 第二节固化为"三大操作" Ingest / Query / Lint（含触发器 + 流程 + 约束）
- 原"Clippings / log / 协作边界"等散落章节折叠进 schema 的对应位置
- 新增 Git 章节、演化历史表
- 仍标 `status/reviewed`（v2 不构成新协议，仅重对齐）

---

## [2026-06-13] apply | 5 篇核心概念笔记落到 01-Concepts/基础理论/

应用了 Inbox/_drafts/concepts/ 下 5 篇概念草案：
- `Tokenization.md`（分词）
- `Embedding.md`（向量表示）
- `MoE.md`（混合专家）
- `采样策略.md`
- `量化.md`

vault 规模：173 → **178 篇**，01-Concepts 26 → 31 篇，基础理论层 4 → 9 篇。
清理 5 个草案 + 1 个 README 共 6 个 Inbox 文件。

---

## [2026-06-13] apply | 5 篇协议笔记落到 04-Frameworks/

应用了 Inbox/_drafts/protocols/ 下 5 篇协议草案：
- `04-Frameworks/MCP 协议/MCP 原理.md`（完整重写）
- `04-Frameworks/MCP 协议/MCP Server 开发.md`（完整重写）
- `04-Frameworks/MCP 协议/ACP 原理.md`（标 `status/archived`）
- `04-Frameworks/AG-UI 协议/AG-UI 原理.md`（补消息类型+流程+代码）
- `04-Frameworks/ANP 协议/ANP 原理.md`（补流程+安全+代码）

协议层从空壳状态变为 vault 中最深的领域之一。
清理 5 个草案 + 1 个 README 共 6 个 Inbox 文件。

---

## [2026-06-13] fix | 修复 LLM 模型地图 10 处断链

老大的 `00-MOC/LLM 模型地图.md` 有 10 处指向不存在笔记的 wikilink：
- 删除 5 处（o3-pro / Grok 3.5 / Grok 3.5 mini / Grok 3 DeepSearch）
- 改写 5 处为真实存在笔记（Gemini 3 / Qwen 3.5 / MiniMax M2.7 / MiMo-V2.5 / MAI-Thinking-1 / Nemotron 3）

顺手补漏 9 处真实存在但原 MOC 没引用的笔记（MiniMax M3、Qwen 3.6、Grok 4.20、Kimi K2.6、GLM-5.1 等）。

---

## [2026-06-13] lint | 全 vault 诊断报告（173 篇 → 178 篇前的快照）

完整诊断见 `Inbox/00-知识库诊断报告 2026-06-13.md`。关键发现：
- 10 处断链（已修复，见上条）
- 5 篇协议笔记空壳（已补全，见上上条）
- 26 篇概念笔记 + 缺 ~15 个关键概念（其中 5 个已新建，见上上上条）
- 2026-02 后 25+ 篇模型笔记联网核查失败（沙箱限制）
- 几乎所有笔记 `status/seed` 应升级

诊断依据：扫描 173 个 .md 文件 + 10 个模板 + 5 个 MOC。

---

## [2026-06-13] query | 老大问"你可以自行补全目前的知识库吗"

本轮会话起点。老大给出三选项：
1. 诊断先行（采纳）
2. 自主度：我起草 → 老大审 → 入库（采纳）
3. 信息源：联网（采纳，但沙箱屏蔽失败）

本日志后续所有操作都基于这三个决策。

---

<!-- 
模板：
## [YYYY-MM-DD] <操作> | <一句话描述>

详细描述（多行），
包括：
- 涉及哪些文件
- 数量变化（如 173 → 178）
- 任何清理（已删除的临时文件）

-->