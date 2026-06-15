---
created: 2026-06-15
tags: [资源, RSS, 信息流, D线]
type: 索引
---

# RSS 订阅清单 · 使用说明

> 配套文件：[[../06-Resources/RSS订阅清单-AI-Agent.opml]]
> 用途：建立 D 线（追踪前沿）的统一信息入口，配合「每日扫描 → Clippings → 每周精读」工作流。

## 一、这个清单解决什么问题

你现在的痛点是 `06-Resources` 和 `Clippings` 几乎空——不是没东西可读，而是**没有一个稳定的信息入口**，结果要么忘了看、要么在 10 个网站间跳来跳去浪费 30 分钟。

这份 OPML 把 Tier 1/2/3 的源全部聚合成一个文件，**导入 RSS 阅读器后，所有更新统一推送到一个收件箱**，每天扫一遍即可。

## 二、导入步骤（Fluent Reader 为例）

1. **装阅读器**：下载 Fluent Reader（开源免费）
   - GitHub: https://github.com/yang991178/Historical-Fluent-Reader/releases
   - 或新版 Fluent Reader: https://github.com/yang991178/fluent-reader/releases
2. **导入 OPML**：
   - 打开 Fluent Reader → 菜单 → `File` → `Import` → `Import from OPML`
   - 选择本库文件 `06-Resources/RSS订阅清单-AI-Agent.opml`
3. **同步**（可选）：Fluent Reader 支持自建 Inoreader/Feedly 同步，前期不用配
4. **验证**：导入后查看每个源是否显示文章列表（见下方验证清单）

> 也支持 Feedly / Inoreader / QuiteRSS / 任意标准 RSS 阅读器，OPML 是通用格式。

## 三、源分类速查

### 🥇 Tier 1 · 核心源（每天扫一遍，~20 分钟）

| 源 | 类型 | 价值 |
|---|---|---|
| arXiv cs.AI / cs.CL / cs.MA / cs.LG | 论文 | 一手论文流，每日更新 |
| Hugging Face Daily Papers | 论文 | 社区投票的精选论文（社区桥接源） |
| Anthropic News/Research/Engineering | 官方 | Agent 最佳实践黄金源（社区桥接源） |
| OpenAI News | 官方 | 模型发布 + 研究 |
| Google DeepMind | 官方 | 前沿研究 |
| Simon Willison | 博客 | LLM 应用前沿最敏锐的个人博主 |
| Lilian Weng | 博客 | 长文综述，一篇顶十篇（月度精读） |

### 🥈 Tier 2 · 高价值源（每周看一次）

| 源 | 类型 | 价值 |
|---|---|---|
| Latent Space（文章+播客） | Newsletter | Agent/Infra 领域最好访谈 |
| The Batch（吴恩达） | Newsletter | 每周 AI 新闻精选 |
| Eugene Yan / Chip Huyen | 博客 | LLM 工程化、评估、部署 |
| Papers with Code | 论文 | 论文 + 配套代码 |

### 🥉 Tier 3 · 中文源（省翻译，做索引）

机器之心、量子位、李宏毅、李沐、宝玉——**只用做快速过滤和导读**，别替代一手资料。

## 四、关于"社区桥接源"的说明

部分源没有官方 RSS，用的是 GitHub 上 `Olshansk/rss-feeds` 项目自动生成的 feed（每小时抓取一次）：
- Anthropic 全系（News / Research / Engineering）
- The Batch

**依赖**：这些 raw.githubusercontent.com 地址需要能访问 GitHub（你有 VPN，没问题）。
**风险**：如果项目停更，这些 feed 会失效——失效时去 https://github.com/Olshansk/rss-feeds 确认最新地址，或用 RSSHub 自建。

## 五、导入后的验证清单

导入后，对照下表检查每个源**是否能加载出文章列表**。如果某个源显示空或报错，按"备用方案"处理：

| 源 | 预期状态 | 备用方案 |
|---|---|---|
| arXiv ×4 | ✅ 必有文章 | 无（官方稳定） |
| HF Papers (takara.ai) | ✅ 有文章 | 失效则手动访问 huggingface.co/papers |
| Anthropic ×3 (社区源) | ✅ 有文章 | 失效则去 Olshansk/rss-feeds 看新地址 |
| OpenAI News | ✅ 有文章 | 无（官方稳定） |
| DeepMind | ✅ 有文章 | 无（官方稳定） |
| Simon Willison | ✅ 有文章 | 无（官方稳定） |
| Lilian Weng | ⚠️ 低频 | 月更，没新文章正常 |
| Latent Space ×2 | ✅ 有文章 | 无 |
| Eugene Yan | ⚠️ 低频 | 月更正常 |
| Chip Huyen | ⚠️ 低频 | 季更，没新文章正常 |
| 机器之心 / 量子位 | ✅ 有文章 | 失效则去官网直接看 |

## 六、和知识库工作流的对接

```
RSS 阅读器（每日扫描 20 分钟）
     │
     ├─ 不相关 → 跳过
     ├─ 有点意思 → Obsidian Web Clipper 存进 Clippings，打 #unread
     └─ 很关键 → 存 + 顶部写"为什么关注"
                    │
                    ▼
         每周精选 1~2 篇 → 精读 → 写进 01-Concepts 或 05-Projects
```

## 七、维护建议

- **每月**：清掉没再更新的源，新增本月发现的好源
- **季度**：重新评估 Tier 归类，太水的源降级或删除
- **保持克制**：源不是越多越好，控制在 **30 个以内**最舒服，否则会被信息淹没

## 八、相关笔记

- [[Agent 全景地图]]
- [[LLM 基础理论地图]]
- [[框架与工具地图]]
