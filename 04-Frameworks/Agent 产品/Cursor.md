---
title: Cursor
aliases: [Cursor, cursor]
vendor: Cursor Inc.
category: AI 代码编辑器
pricing: Pro $20/月, Business $40/月
url: https://cursor.com
tags:
  - framework
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-12
---

# Cursor

> AI-first 代码编辑器（VS Code fork），内置 Agent 模式。支持多文件编辑、代码库问答、Composer 多文件重构。

## 基本信息

| 项目 | 详情 |
|------|------|
| 厂商 | Cursor Inc. |
| 官网 | https://cursor.com |
| 发布日期 | 2023 |
| 定价 | Pro $20/月, Business $40/月 |
| 目标用户 | 前端/全栈开发者 |
| 底层模型 | Claude Sonnet 4, GPT-4o, 自研模型（可切换） |

## 核心能力

1. Agent 模式：自动执行多步编码任务
2. Composer：跨文件重构
3. Tab 补全：上下文感知的代码补全
4. @ 符号引用文件、文档、代码库

## 工作流

```
用户在 Cursor IDE 中编码
  ├─ Tab 补全 → 行级代码建议
  ├─ Cmd+K → 内联编辑
  ├─ Composer → 多文件重构
  ├─ Agent 模式 → 自动执行终端命令、文件操作
  └─ Chat → 代码库问答
```

## 技术架构

- **基础**: VS Code fork，继承完整扩展生态
- **模型**: 多模型切换（Claude, GPT-4o, 自研）
- **索引**: 本地代码库嵌入索引，支持语义搜索
- **上下文**: @ 引用系统（文件、文档、codebase）

## 与同类对比

| 特性 | Cursor | Windsurf | Claude Code |
|------|--------|----------|-------------|
| 优势 | VS Code 生态、Tab 补全体验好 | 流式 Cascade | 终端原生、Subagent |
| 劣势 | Agent 深度有限 | 扩展生态小 | 无 GUI |

## 最佳实践

<!-- 待填写 -->

## 已知局限

<!-- 待填写 -->

## 使用心得

<!-- 个人使用体验、踩坑记录 -->

## 参考资料

- [Cursor 官网](https://cursor.com)
