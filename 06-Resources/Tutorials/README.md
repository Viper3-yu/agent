---
title: Tutorials 教程索引
aliases: [教程笔记, Tutorial Notes, Hands-on Notes]
tags:
  - MOC
  - status/reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Tutorials 教程索引

> 我**亲手跑过**的实操教程：官方 Quickstart、Hackathon 示例、Bilibili / YouTube 实操、Notebook 复现等。

## 命名约定

- 文件名 = 教程主题（kebab-case）
- 例：`langgraph-quickstart.md`、`mcp-server-build.md`
- 必填 `hands_on: true` 标签（区分"看了"与"做了"）
- `time_cost` 字段记实操耗时（重要 ROI 参考）

## 必填 frontmatter

```yaml
---
title: <教程标题>
tags:
  - tutorial
  - status/reviewed
hands_on: true                  # 必填：跑过不是看过
time_cost: 2h                   # 必填：实际耗时
difficulty: beginner/intermediate/advanced
source_url: https://...
repo: https://github.com/...     # 涉及代码时
---

## 目标
## 步骤（带截图/链接）
## 踩坑记录
## 复现命令
## 我的扩展
```

## 当前教程

<!-- 按主题分组 -->

### LLM 基础
- （无）

### Agent 框架
- （无）

### RAG / 检索
- （无）

### 微调 / 部署
- （无）

## 添加新教程

```bash
# 1. 把官方文档 / 视频 / notebook 链接记入 Clippings/
# 2. 跑通最小可复现案例
# 3. 写教程笔记到 06-Resources/Tutorials/<主题>.md
# 4. 在本 README 对应主题分组下加 wikilink
```

## 与 04-Frameworks 的区别

- **04-Frameworks/<某框架>.md** = 框架本身的深度笔记（持续更新）
- **06-Resources/Tutorials/<某教程>.md** = 跟着某个教程**实操的记录**（一次性的）

## 相关资源

- → [[../README|Resources 层根]]
- → [[../Articles/README|文章索引]]
- → [[../Courses/README|系统课程]]
