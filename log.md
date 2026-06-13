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

## [2026-06-13] chore | 初始化本地 git 仓库

执行 [[../00-MOC/KB 构建规范#九、Git 版本管理|KB 构建规范 §9]] 实施步骤的前半段：
- `git init -b main`
- 写入 .gitignore（**调整**：忽略整个 `.obsidian/plugins/` 41M + `.obsidian/themes/` 1.1M，仅追踪 7 个配置 JSON）
- 首次 commit `cf50c3c`：209 文件 / 18085 行 / `.git` 941K

vault 净变化：197 个 .md 笔记 + 7 个 .obsidian 配置 = 209 tracked。

---

## [2026-06-13] chore | 推送 GitHub 成功

老大提供仓库 URL：`https://github.com/Viper3-yu/agent.git`
- 远端有 1 个初始 commit（老大手写 README："这是obsidian 的agent知识库"）
- 本地 rebase 把老大 README 留作根，3 个本地 commit 接到上面
- `git push -u origin main` 成功：`b4521b8..8d64b33`

**过程中补的 .gitignore**：
- `.claudian/`（Claudian 会话日志，3029 行删除后忽略）

**远端地址**：https://github.com/Viper3-yu/agent

---

## [2026-06-13] docs | 覆盖 README 为专业版（B 方案）

老大决定 B：保留根 commit `b4521b8` 不动，覆盖 README 内容为专业版：
- 三层架构描述（Raw / Wiki / Schema）
- 目录速览 + 怎么用 + 维护说明
- Commit 约定 + Karpathy 模式参考链接

老大手写原文"这是obsidian 的agent知识库"被新 README 替代，但 commit 仍作为根保留。

**学到的沙箱行为**：`github.com:443` 间歇性屏蔽，4-5 次 push 中 1-2 次能通——下次卡住多试几次。

最终远端：`a5ad935..df4e59c` 推送成功，5 commits。

---

## [2026-06-13] chore | 装 obsidian-git + 修配置

老大装好 obsidian-git（社区插件），观察到三件事：

1. **`.obsidian/graph.json` 应被 gitignore**——它是 Obsidian 图形视图 per-device 状态，每次开 Obsidian 都变
   → 加入 .gitignore，从 tracking 移除

2. **15 个 copilot 文件行尾 CRLF/LF 噪音**——rebase 时 Windows 自动转换
   → 新建 `.gitattributes` 锁定 `* text=auto eol=lf`
   → `git add --renormalize .` 一次性把所有文本文件规范化

3. **`community-plugins.json` 正确添加了 `"obsidian-git"`**——gitignore 已让插件二进制不进 git，符合设计

老大后续可按推荐设置配置 obsidian-git（5 分钟自动 commit + push）。

---

## [2026-06-13] fix | 修复孤立笔记 14% → 2%

老大观察到大量笔记之间无关联，调查发现 25/173 (14%) 是孤立笔记。修复：

**根因分类**（25 → 4）：
| 类别 | 数量 | 处理 |
|------|------|------|
| 故意独立 | 3（README/欢迎/测试） | 保留 |
| 占位 | 8（05-Projects/06-Resources 下的占位） | 删除 |
| MOC 缺入链 | 4（4 个 MOC 没人链到） | 加互链 + 重写欢迎 |
| 真实笔记未挂 MOC | 10 | 补到对应 MOC |

**变更**：
- 新建 `00-MOC/LLM 基础理论地图.md` 收容 5 个新概念笔记
- 重写 `欢迎.md` 为 vault 入口（链所有 5 个 MOC）
- 4 个 MOC 顶部加互相导航条
- LLM 模型地图补 3 条（Gemini 3.1 Pro / Flash-Lite / GPT-5.4-mini-nano）
- Agent 全景地图补 5 条（Perplexity / Windsurf / Claude Code 命令参考 / LlamaIndex / CrewAI Flow）
- 删除 8 个空占位笔记

**效果**：
- vault 笔记数：173 → 166
- 孤儿率：14% → 2%
- 5 个 MOC 互链 + 都有入链

---

## [2026-06-13] meta | 新建 Graph 视图配置指南

老大问"图谱都是一种颜色"——根因是 `.obsidian/graph.json` 的 `colorGroups: []`。

新建 `00-MOC/Graph 视图配置.md` 给出：
- 8 组按目录路径的颜色映射（path:00-MOC 链→金 / 01-Concepts→紫 / ...）
- 按 topic 标签的进阶配色
- Juggl 插件安装 + 4 种着色维度（degree / closeness / community / tag）

老大在 Obsidian UI 里手动设（graph.json 是 per-device，Claudian 不能直接改）。

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