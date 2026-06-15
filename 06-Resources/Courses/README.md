---
title: Courses 课程索引
aliases: [课程笔记, Course Notes, MOOC]
tags:
  - MOC
  - status/reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Courses 课程索引

> 完整的**系统性课程**笔记（CS224N / CS336 / DeepLearning.AI / Hugging Face / Bilibili 系列课等），区别于碎片化的 Tutorials。

## 命名约定

- 文件名 = 课程简称（kebab-case）
- 例：`cs224n-2024.md`、`stanford-cs336-2025.md`、`huggingface-agents.md`
- 必填 `progress: 0-100%` 标签
- 必填 `hours_total` 和 `hours_done`

## 必填 frontmatter

```yaml
---
title: <课程全名>
aliases: [<课程代号>]
tags:
  - course
  - status/draft | reviewed
provider: <学校 / 平台>
year: 2024
instructor: <老师>
url: https://...
hours_total: 60
hours_done: 12
progress: 20%                  # 派生字段，但可手填
---

## 课程目标
## 章节结构
## 我的学习顺序
## 关键笔记（链接到子笔记）
## 习题 / 项目
## 完成度
```

## 当前课程

<!-- 按状态分：进行中 / 已完成 / 已放弃 -->

### 进行中
- （无）

### 已完成
- （无）

### 已放弃 / 待重启
- （无）

## 添加新课程

```bash
# 1. 在 Clippings/ 记录课程链接 + 简介
# 2. 创建课程笔记到 06-Resources/Courses/<代号>.md
# 3. 每章单独一个子笔记
mkdir "06-Resources/Courses/<代号>"
touch "06-Resources/Courses/<代号>/<章节>.md"
# 4. 在课程笔记"章节结构"下用 wikilink 串起来
```

## 子笔记结构（每章）

```markdown
## 学习日期
## 视频 / 课件时间戳
## 关键概念
## 我之前的理解 vs 现在
## 习题 / 编程作业
## 卡点
```

## 与 Tutorials 的区别

- **Tutorials** = 单个**实操**任务（1~2h 完成）
- **Courses** = 完整**课程体系**（10h+ 跨度）
- 一门课程可拆出多个 Tutorials 作为子任务

## 相关资源

- → [[../README|Resources 层根]]
- → [[../Tutorials/README|教程索引]]
- → [[../Papers/README|相关论文]]
