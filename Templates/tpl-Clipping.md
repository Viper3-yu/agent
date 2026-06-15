---
title: <% tp.file.title %>
aliases: [<% tp.file.title %>]
tags:
  - clipping
  - status/seed
  - meta/unread
created: <% tp.date.now("YYYY-MM-DD") %>
updated: <% tp.date.now("YYYY-MM-DD") %>
source: 
captured: <% tp.date.now("YYYY-MM-DD") %>
author: 
type: <article | paper | video | podcast | tweet | code>
---

# <% tp.file.title %>

> 一句话摘要（必填）。看完摘要就知道这篇文章是否值得精读。

## 来源

- **URL**：<!-- 必填 -->
- **作者 / 发布方**：<!-- 如 Lilian Weng / Anthropic Engineering -->
- **发布日期**：YYYY-MM-DD
- **抓取日期**：<% tp.date.now("YYYY-MM-DD") %>
- **抓取方式**：<!-- Web Clipper / 手动 / RSS 阅读器 -->

## 标签

<!-- 至少 2-4 个，便于后续检索。例： -->
<!-- - topic/agent -->
<!-- - topic/eval -->
<!-- - meta/high-priority -->

- `topic/`
- `#unread`

## 为什么关注

<!-- 一段话，写"这篇为什么值得读"。例： -->

<!-- - Anthropic 第一次公开 multi-agent research system 的架构细节 -->
<!-- - 复现 [[../05-Projects/2026-06-repro-building-effective-agents/README]] 可能用得到 -->

## 关键要点（3-5 条）

<!-- 精读时填。一开始可以不写，等精读时再补。 -->

1.
2.
3.
4.
5.

## 原文摘录（可选）

<!-- 直接 copy 关键段落，附原文位置 -->

> ... — <段落在原文的位置>

## 行动项（精读后必填）

<!-- 读完后的下一步： -->

- [ ] 写 vault 笔记 → 路径：
- [ ] 归档到 06-Resources/ 对应清单
- [ ] 转发给老大参考
- [ ] 暂不处理（理由：）

## 反向链接

<!-- 后续从 vault 笔记 / MOC 反链到这里 -->

- [[]]

---

<!--
模板说明（删除本段后保存）：

1. **Clipping 的最低要求**：URL + 日期 + 一句话摘要 + 2-4 个标签
2. **不要让 Clipping 变垃圾堆**：每周清理一次 #unread 已读/已处理的
3. **精读后填"行动项"**：决定这篇是写笔记 / 归档 / 跳过
4. **meta/unread 状态**：精读完成后改成 meta/read，并去掉 #unread
-->
