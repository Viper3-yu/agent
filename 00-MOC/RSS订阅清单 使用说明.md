---
created: 2026-06-16
updated: 2026-06-16
tags: [资源, RSS, 信息流, D线]
type: 索引
---

# RSS 订阅清单 · 使用说明

> 配套文件：[[RSS订阅清单-AI-Agent.opml]]
> 用途：建立 D 线（追踪前沿）的统一信息入口，配合「每日扫描 → Clippings → 每周精读」工作流。
> 最后验证：2026-06-16

## 一、这个清单解决什么问题

知识库的痛点是 `06-Resources` 和 `Clippings` 内容偏少——不是没东西可读，而是**没有一个稳定的信息入口**。这份 OPML 把核心源聚合成一个文件，**导入 RSS 阅读器后，所有更新统一推送到一个收件箱**，每天扫一遍即可。

## 二、导入步骤（Fluent Reader）

1. **装阅读器**：Fluent Reader（开源免费）
   - https://github.com/yang991178/fluent-reader/releases
2. **导入 OPML**：
   - 打开 Fluent Reader → 菜单 → `File` → `Import` → `Import from OPML`
   - 选择本库文件 `06-Resources/RSS订阅清单-AI-Agent.opml`
3. **同步**（可选）：支持自建 Inoreader/Feedly 同步，前期不用配

> OPML 是通用格式，Feedly / Inoreader / QuiteRSS 等任意标准 RSS 阅读器都支持。

## 三、关于"导入报错"的说明（重要）

导入时可能弹出"导入 N 项订阅源时出错"，分两类处理：

### 3.1 提示"已存在 / duplicate"（非错误）
表示该源之前已导入过，**直接忽略**，不影响使用。

### 3.2 真正的报错
本版 OPML 已经移除了所有实测不可访问的源（见下方变更说明），**剩余带 `xmlUrl` 的源都是可用 RSS**。如果仍然报错，通常是：
- **VPN 未开或代理未生效** → Fluent Reader 需要走系统代理，确认 VPN 是"全局/TUN 模式"而不是 PAC
- **raw.githubusercontent.com 被 DNS 污染** → 本版已移除所有该域名的源
- **网络瞬时超时** → 右键该源 → `Refresh` 重试

## 四、本次修订变更（2026-06-16）

上一版 OPML 导入时有 8 个源报错，已全部处理：

| 源 | 旧状态 | 处理 |
|---|---|---|
| Anthropic News/Research/Engineering | ❌ raw.githubusercontent 不稳定 | 移除，改为手动关注 anthropic.com/news |
| The Batch（吴恩达） | ❌ 同上 | 移除，改为订阅官方邮件 |
| Eugene Yan | ❌ 站点无原生 RSS（404） | 移除，改为定期访问 /writing/ |
| Papers with Code | ❌ 无原生 RSS | 移除，改为定期访问 /latest |
| 量子位 | ❌ /feed 不稳定 | 移除 RSS，保留官网手动访问 |
| 机器之心 | ✅ 实测可用 | 保留 `jiqizhixin.com/rss` |

**移除的源不是被放弃，而是改用"手动关注 / 邮件订阅 / 定期访问"** —— 因为它们没有稳定可达的 RSS。这样导入时不会再有报错。

## 五、源分类速查（与新版 OPML 对应）

### 🥇 Tier 1 · 核心源（每天扫一遍，~20 分钟）

**带 RSS 的源（导入即用）：**

| 源 | 类型 | 备注 |
|---|---|---|
| arXiv cs.AI / cs.CL / cs.MA / cs.LG | 论文 | 官方稳定，每日更新 |
| Hugging Face Daily Papers | 论文 | 社区桥接源 takara.ai |
| OpenAI News | 官方 | 官方 RSS 稳定 |
| Google DeepMind | 官方 | blog.google feed |
| Simon Willison | 博客 | 个人域名，最敏锐 |
| Lilian Weng | 博客 | 月更，长文综述 |

**手动关注（无 RSS）：**
- Anthropic（news / research / engineering）→ 定期访问官网

### 🥈 Tier 2 · 高价值源（每周看一次）

**带 RSS：** Latent Space 文章 + 播客、Chip Huyen
**手动关注：** The Batch（邮件）、Eugene Yan（/writing/）、Karpathy（YouTube）、Papers with Code（/latest）

### 🥉 Tier 3 · 中文源

**带 RSS：** 机器之心
**手动关注：** 量子位、李宏毅、李沐、宝玉

## 六、导入后的验证清单

导入后检查每个带 RSS 的源是否能加载出文章列表：

| 源 | 预期 | 备注 |
|---|---|---|
| arXiv ×4 | ✅ 必有文章 | 官方稳定 |
| HF Papers (takara.ai) | ✅ 有文章 | 社区源，偶尔慢 |
| OpenAI News | ✅ 有文章 | 官方稳定 |
| DeepMind | ✅ 有文章 | 官方稳定 |
| Simon Willison | ✅ 有文章 | 几乎日更 |
| Lilian Weng | ⚠️ 低频 | 月更，空属正常 |
| Latent Space ×2 | ✅ 有文章 | 稳定 |
| Chip Huyen | ⚠️ 低频 | 季更，空属正常 |
| 机器之心 | ✅ 有文章 | 已实测可用 |

## 七、和知识库工作流的对接

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

## 八、想让被移除的源重新进 RSS？

如果你希望 Anthropic / The Batch / Eugene Yan / Papers with Code / 量子位也能在 RSS 里收到，**唯一稳定的办法是自建 RSSHub**：

1. 参考 RSSHub 文档：https://docs.rsshub.app
2. 本地或服务器部署一个 RSSHub 实例
3. 用 `http://你的实例地址/anthropic/news` 这类路由订阅

前期不建议折腾，**手动关注 + 定期访问完全够用**。等你跑顺 RSS 工作流 1~2 个月，觉得值得自动化了再上。

## 九、维护建议

- **每月**：清掉没再更新的源，新增本月发现的好源
- **季度**：重新评估 Tier 归类，太水的源降级或删除
- **保持克制**：源控制在 20~30 个以内最舒服，否则会被信息淹没

## 十、相关笔记

- [[论文清单]]
- [[书单与课程]]
- [[awesome 清单汇总]]
- [[Agent 全景地图]]
- [[框架与工具地图]]
