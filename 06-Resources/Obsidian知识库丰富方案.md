---
created: 2026-06-15
updated: 2026-06-15
tags: [方案, 知识库, 信息流, 索引, status/reviewed]
type: 方案文档
status: 草稿 → 已审（§5 全部采纳）
---

# Obsidian 知识库丰富方案

> 目标：把当前"骨架硬、肌肉少"的理论型知识库，升级为「深度消化 + 追踪前沿」双轨并行的活的知识库。
> 适用：AI / Agent 领域个人知识库，每天可投入 30~60 分钟，中英混看，有稳定 VPN。

---

## 一、现状诊断

### 1.1 当前结构

库位置：`C:\Users\25868\obsidian1`，采用 PARA + MOC 混合结构，主题聚焦 AI / Agent。

| 文件夹 | 文件数 | 状态 |
|---|---|---|
| 00-MOC | 9 | ✅ 索引地图完整（Agent 全景、LLM 理论、模型、框架等） |
| 01-Concepts | 31 | ✅ 概念骨架扎实（Agent 架构 / 能力 / 基础理论） |
| 02-Models | 61 | ✅ 模型笔记量大 |
| 03-Prompt | 12 | ⚠️ 偏少 |
| 04-Frameworks | 46 | ✅ 框架/工具非常全（LangGraph、CrewAI、MCP、Claude Agent SDK…） |
| **05-Projects** | **0** | ❌ **完全空** —— 缺实战案例/项目复盘 |
| **06-Resources** | **2** | ❌ **几乎空** —— 缺论文/书单/课程索引 |
| **Clippings** | **1** | ❌ **几乎空** —— 缺一手内容剪藏 |
| Inbox | 7 | 待整理 |
| Templates | 10 | ✅ 有模板基础 |

### 1.2 一句话诊断

**"骨架很硬、肌肉偏少"的理论型知识库**——概念和工具维度很全，但缺实战项目、一手资源、有血有肉的案例。`05-Projects` 全空、`06-Resources` 和 `Clippings` 几乎空，是最明显的"待长肉"的地方。

### 1.3 两个核心目标

- **A 线（深度消化）**：把 AI/Agent 学透，建立体系化理解 → 需要"慢"
- **D 线（追踪前沿）**：跟上快速演进的最新进展 → 需要"快"

**内在张力**：A 需要"慢"（读经典、啃论文、复现、写长笔记），D 需要"快"（高频扫描、快速过滤）。很多知识库失败的根因就是没分开处理这两种节奏，结果要么被新信息淹没（D 淹了 A），要么沉淀太久脱离前沿（A 脱了 D）。

---

## 二、核心策略：双轨 + 中转

给 A 线和 D 线分别设计不同的资料源 + 工作流，再用一个"中转层"连接。

```
[前沿信息源] ──每日扫描──▶ [Inbox 快速过滤] ──每周筛选──▶ [深度消化 → Concept]
        (D 线，快)                    │                       (A 线，慢)
                                      ▼
                              [Clippings 剪藏]
                              [Resources 索引]
```

### 2.1 三条纪律

1. **D 线的内容不许直接污染 Concept**——必须经过"值得精读吗"这道闸门
2. **D 线绝大多数内容只存索引**（Clippings / Resources），不深度处理
3. **A 线每周只精读 1~2 篇**——宁少勿滥，吃透比看多重要

这个分工直接解决 `06-Resources` / `Clippings` 几乎空、`05-Projects` 全空的问题——因为它们恰恰是 D 线的产物 + A 线的承接位。

---

## 三、资料源清单（分四档）

> **优先级原则**：时间有限，先把精力集中在 Tier 1，跑顺了再加 Tier 2。
> **所有源已验证可访问**（有稳定 VPN 前提下）。

### 🥇 Tier 1 · 最高优先级，日常必看（强烈推荐先建立）

这是"投入产出比最高"的核心圈，覆盖 A+D 两个目标。

#### 3.1 arXiv 每日 AI 论文（D 线核心）

- `arxiv.org/list/cs.AI/recent`（人工智能）
- `arxiv.org/list/cs.CL/recent`（NLP / LLM）
- `arxiv.org/list/cs.MA/recent`（多智能体）
- `arxiv.org/list/cs.LG/recent`（机器学习）
- **怎么用**：每天扫标题 + 摘要，5 分钟。看到与 Agent / LLM 主题相关的存到 Clippings，加 `#paper` 标签 + 一句话为什么关注

#### 3.2 Hugging Face Daily Papers（A+D，强推）

- `huggingface.co/papers`
- 社区投票的"今日好论文"，比 arXiv 全量更精，**最适合每天 30 分钟节奏**
- ⚠️ HF 2024 年起国内被墙，需 VPN

#### 3.3 Anthropic / OpenAI / Google DeepMind 官方博客（A 线核心）

- `anthropic.com/news`（尤其 Engineering 和 Research 分类）
- `openai.com/research`
- `deepmind.google/discover/blog/`
- **为什么必看**：库重点在 Agent，而 Agent 的"一手最佳实践"基本来自这三家。Anthropic 的 *"Building effective agents"*、*"Multi-agent research system"* 这类文章是黄金素材，应直接精读进 Concept

#### 3.4 Simon Willison's Weblog（D 线神器）

- `simonwillison.net`
- 每天/每两天一篇，**追踪 LLM 应用前沿最敏锐的个人博主**，尤其关注 Agent、工具调用、评测。信息密度极高，必订阅

#### 3.5 Lilian Weng's 博客（A 线镇库）

- `lilianweng.github.io`
- OpenAI 前研究主管，长文综述质量极高（"LLM Agent"、"RAG"、"Reward Hacking"...）。一篇顶十篇，适合月度精读

### 🥈 Tier 2 · 高价值，每周看一次

| 源 | 链接 | 价值 |
|---|---|---|
| **Latent Space**（播客 + Newsletter） | `latent.space` | Agent / Infra 领域最好的访谈节目，逐字稿很值 |
| **The BATCH**（吴恩达） | `deeplearning.ai/the-batch` | 每周一次的 AI 新闻精选，省时间 |
| **Andrej Karpathy** 的 YouTube / X | `youtube.com/@AndrejKarpathy` | 讲 LLM 原理最好的入门到进阶内容 |
| **Eugene Yan** | `eugeneyan.com` | 偏 LLM 系统工程、评估、部署，补"实战肌肉" |
| **Chip Huyen** | `huyenchip.com` | 同上，LLM 工程化 |
| **Papers with Code** | `paperswithcode.com` | 找论文 + 配套代码 |

### 🥉 Tier 3 · 中文圈优质源（中英混看的好补充）

**只用做"索引 / 导读"**，别拿它替代一手资料——它帮你快速过滤 + 省翻译时间。

| 源 | 链接 | 用途 |
|---|---|---|
| **机器之心 / 量子位** | `jiqizhixin.com` / `qbitai.com` | 新闻快，适合 D 线扫描标题 |
| **李宏毅 ML 课程** | `speech.ee.ntu.edu.tw`（B 站有） | 中文讲 LLM 原理最好，A 线补基础 |
| **宝玉的科技分享** | X `@dotey` / `baoyu.io` | 高质量 AI 资讯编译，省翻译 |
| **跟李沐学 AI** | B 站搜"跟李沐学AI" | 沐神的论文带读是中文圈最好的 A 线素材之一 |

### 🔧 Tier 4 · 工具辅助层（必装，省大量时间）

这几个不是"读的源"，是**让 30 分钟跑得动的关键工具**：

| 工具 | 费用 | 价值 |
|---|---|---|
| **Obsidian Web Clipper** | 🆓 免费（官方开源） | 一键存网页到 Clippings，带元数据。**D 线的命脉** |
| **RSS 阅读器**（Fluent Reader） | 🆓 免费开源 | 统一入口扫描所有源，否则会在 10 个网站间跳来跳去浪费 30 分钟 |
| **沉浸式翻译**（Immersive Translate） | 🆓 基础免费 | 中英混看神器，网页 / PDF 双语对照 |
| **Readwise Reader**（可选） | 💰 约 $8/月 | 专门做"稍后读 + 高亮同步到 Obsidian"，预算能上再考虑 |

---

## 四、落地节奏（适配每天 30~60 分钟）

### 4.1 每日（~20 分钟）—— D 线扫描

```
打开 RSS 阅读器（统一入口）
  ├─ 扫标题，3 秒一篇判断：
  │   ├─ 不相关 → 跳过
  │   ├─ 有点意思 → Web Clipper 存进 Clippings，打 #unread
  │   └─ 很关键 → 存 + 在笔记顶部写一句话"为什么关注"
  └─ arXiv / HF Papers 扫一遍（5 分钟）
目标：30 分钟内完成，不留债
```

### 4.2 每周（~1 小时，挑一天）—— A 线消化 + 中转

```
1. 清 Inbox + 处理本周 Clippings（20 分钟）
   - 把"其实是概念"的移进 01-Concepts 或 06-Resources
   - 把"其实是项目"的移进 05-Projects
2. 精读 1~2 篇本周最佳（40 分钟）
   - 用"卡片笔记法"：一篇论文/文章 → 提炼 3~5 张原子笔记
   - 在对应 MOC 加一条链接
3. 更新一次相关 MOC（10 分钟）
```

### 4.3 每月（~1.5 小时）—— 复盘 + 重构

```
1. 回顾本月新增笔记，做一次"链接编织"（看 Graph 视图，补内部链接）
2. 重写 1~2 个 MOC，让结构更清晰
3. 删/归档本月没再用过的 Clippings（保持新鲜度）
```

---

## 五、知识库具体改造建议（针对现状）

### 5.1 激活 `05-Projects`（最大空洞）

`05-Projects` 全空是最大的问题。**A 线的深度消化，没有实战项目就立不住**——你自己跑过的东西，笔记才记得牢、才真正"是你的"。

建议立刻起 1~2 个项目笔记，例如：
- 「复现 Anthropic 的 *Building effective agents*」
- 「用 LangGraph 跑通一个多智能体 demo」
- 「用 MCP 协议搭一个工具调用 Agent」

### 5.2 建立 `06-Resources` 的"三大索引"

现在只有 2 个文件，建议补：

- `论文清单.md`——按主题分组（Agent 架构 / RAG / 评估 / 对齐安全…）
- `书单与课程.md`——Karpathy、李宏毅、动手学深度学习、Chip Huyen《Designing ML Systems》…
- `awesome 清单汇总.md`——收集 GitHub 上的 awesome-xxx 仓库链接

### 5.3 给 Clippings 定个模板

防止它变成"垃圾堆"。每条剪藏至少有：

- 来源 URL
- 日期
- 一句话摘要
- 2~4 个标签

Templates 文件夹已有 10 个模板，加一个 `Clipping 模板` 即可。

### 5.4 强化 `03-Prompt`

当前只有 12 个文件，偏少。Prompt 工程是 Agent 的基本功，建议补充：
- 经典 Prompt 技巧（CoT、Few-shot、Self-consistency…）
- 实际跑过、验证有效的 Prompt 模板
- 不同模型（GPT / Claude / Gemini）的 Prompt 差异笔记

---

## 六、费用总览

| 项目 | 费用 |
|---|---|
| 全部信息源（arXiv / HF / Anthropic / OpenAI / 各博客…） | 🆓 免费 |
| Obsidian Web Clipper | 🆓 免费 |
| Fluent Reader | 🆓 免费开源 |
| 沉浸式翻译（基础版） | 🆓 免费 |
| Readwise Reader（可选） | 💰 约 $8/月，非必需 |
| **第一步全部成本** | **0 元** |

---

## 七、VPN 与访问说明

有稳定 VPN 前提下，所有源可用。各源访问情况：

### 🔴 需要 VPN

- **Hugging Face**（2024 年起被墙）
- **OpenAI 官网 / 博客**
- **Google DeepMind 博客**（Google 全线）
- **YouTube**（Karpathy 等）→ B 站有人搬运作为备用
- **X / Twitter** → 宝玉等中文博主会编译重要动态

### 🟢 基本能直接访问

- arXiv（多数时候能开，偶尔慢）
- GitHub
- Simon Willison 博客（个人域名站）
- Anthropic 博客（多数时候能开）
- Papers with Code
- 全部中文源（机器之心、量子位、B 站、李宏毅、沐神、宝玉）

---

## 八、第一步可立刻执行（10 分钟）

如果信息量太大，**今天只做这两件事**就能让 D 线"通水"：

1. **装 Obsidian Web Clipper** + **Fluent Reader**
2. **导入 OPML 订阅文件**（见配套文件 [[RSS订阅清单-AI-Agent.opml]]）

跑一周后 Clippings 开始有内容了，A 线自然就有精读素材了。

---

## 九、配套文件

- [[RSS订阅清单-AI-Agent.opml]] —— 22 个已验证源，一键导入 RSS 阅读器
- [[RSS订阅清单 使用说明]] —— 导入步骤 + 验证清单 + 工作流对接

---

## 十、相关 MOC

- [[Agent 全景地图]]
- [[LLM 基础理论地图]]
- [[Prompt Engineering 地图]]
- [[框架与工具地图]]
- [[项目看板]]
