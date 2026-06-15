---
title: People 关键人物索引
aliases: [关键人物, People Notes, Influencers]
tags:
  - MOC
  - status/reviewed
created: 2026-06-15
updated: 2026-06-15
---

# People 关键人物索引

> 在 LLM / Agent 领域持续追踪其输出的**关键人物**笔记。不是简历，是"为什么关注他 / 她 + 我从他学到了什么"。

## 命名约定

- 文件名 = 姓名（PascalCase 或 kebab-case）
- 例：`Andrej Karpathy.md`、`Lilian Weng.md`、`Simon Willison.md`
- frontmatter `role` 字段标明身份
- `tracking_level` 字段表明关注深度

## 必填 frontmatter

```yaml
---
title: <姓名>
aliases: [<英文名>, <中文译名>]
tags:
  - people
  - status/reviewed
role: researcher / engineer / founder / educator
org: <当前机构>
tracking_level: high | medium | low   # 关注深度
homepage: https://...
x_handle: @...
github: https://github.com/...
---

## 一句话定位
<!-- 他 / 她在领域里解决什么问题？ -->

## 我为什么关注
<!-- 1-3 个具体理由 -->

## 持续追踪的产出渠道
<!-- 博客 / X / GitHub / 播客 / 论文 -->

## 我从他 / 她学到的（按时间倒序）
<!-- 日期 + 一条具体收获 -->

## 关键代表作（≤5 个）
- [[../../Clippings/<filename>|<产出名>]]
```

## 当前追踪

<!-- 按 tracking_level 排序 -->

### High（每条产出必读）
- （无）

### Medium（每周扫一遍）
- （无）

### Low（季度回顾）
- （无）

## 推荐关注的 20+ 人物（待老大选择）

| 姓名 | 角色 | 为什么 |
|------|------|-------|
| Andrej Karpathy | 研究员 / 教育者 | 深度讲解 + 实操视频 |
| Lilian Weng | 研究员（前 OpenAI） | 长篇博客覆盖 LLM 几乎所有主题 |
| Simon Willison | 独立开发者 | 实战派 + 早期采用者 |
| Sebastian Raschka | 工程师 / 作家 | 《Build a Large Language Model (From Scratch)》 |
| Yannic Kilcher | 研究员 / 教育者 | 论文讲解视频 |
| Sebastian Bubeck | 微软研究院 | 1-bit LLMs (BitNet) |
| Tri Dao | Together / Stanford | FlashAttention 作者 |
| Andrej / Sasha Rush | HuggingFace | 推理优化 |
| François Chollet | Google / Keras | ARC-AGI / 深度学习评测 |
| Jim Fan | NVIDIA | 具身 Agent / 基础模型 |
| Yang Song | OpenAI | Diffusion / Rectified Flow |
| 苏剑林 (Su Jianlin) | 独立研究者 | RoPE / 苏神 |
| 贾佳亚 (Jia Jiaya) | 港中文 | 视觉基础模型 |
| 90 门徒们 (Cohere, etc) | 创业者 | RAG / Tool Use |
| ... | ... | ... |

## 添加新人物

```bash
# 1. 确认我确实长期从他 / 她学东西（至少 3 个产出）
# 2. 创建人物笔记
touch "06-Resources/People/<姓名>.md"
# 3. 填 frontmatter + 上面 4 段
# 4. 在本 README "当前追踪" 对应 level 下加 wikilink
```

## 与 06-Resources 其他目录的区别

- **Papers** = 一次性论文
- **People** = 持续追踪人
- 一篇论文 → 找作者 → 加到 People

## 相关资源

- → [[../README|Resources 层根]]
- → [[../Articles/README|文章索引]]
- → [[../Papers/README|论文索引]]
