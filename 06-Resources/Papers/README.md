---
title: Papers 论文索引
aliases: [论文笔记, Paper Notes]
tags:
  - MOC
  - paper
  - topic/llm
  - status/reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Papers 论文索引

> LLM / Agent 主题**学术论文**的结构化笔记。论文元数据 + 核心贡献 + 方法 + 实验 + 我的思考。

## 命名约定

- 文件名 = 论文标题的简短版本（小写、连字符）
- 例：`flash-attention.md`、`chain-of-thought.md`、`mamba.md`
- frontmatter `aliases` 写完整论文标题
- `url` 填 arXiv 链接或会议官方页

## 当前论文

<!-- 按主题分组，2017→2026 倒序 -->

### 基础模型
- （无）

### Agent / RLHF / 对齐
- （无）

### RAG / 检索
- （无）

### 多模态
- （无）

### 推理 / CoT
- （无）

## 添加新论文

```bash
# 1. 把 PDF 或 markdown 放到 Clippings/（可选）
# 2. 复制模板
cp Templates/tpl-论文笔记.md "06-Resources/Papers/<论文短名>.md"

# 3. 填 frontmatter 必填字段
#   authors, year, venue, url

# 4. 填"核心贡献/方法/实验结果"三段
# 5. 在本 README 对应主题下加 wikilink
```

## 必填 frontmatter 字段

```yaml
---
title: <论文标题>
authors: [<作者1>, <作者2>]      # 必填
year: 2024                      # 必填
venue: arXiv / NeurIPS / ICML  # 必填
url: https://arxiv.org/...      # 必填
---
```

## 相关资源

- → [[../../Templates/tpl-论文笔记|论文笔记模板]]
- → [[../README|Resources 层根]]
- → [[../微调/微调总览|相关：微调资源]]
- → [[../../Clippings/llm-wiki|Clippings 原始资料]]
