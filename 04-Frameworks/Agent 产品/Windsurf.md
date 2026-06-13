---
title: Windsurf
aliases: [Windsurf, windsurf]
vendor: Codeium
category: AI 代码编辑器
pricing: Pro $15/月, Teams $30/月
url: https://codeium.com/windsurf
tags:
  - framework
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-12
---

# Windsurf

> Codeium 推出的 AI 编辑器，主打 Cascade 流式 Agent 体验。自动理解代码库上下文，逐步执行复杂任务。

## 基本信息

| 项目 | 详情 |
|------|------|
| 厂商 | Codeium |
| 官网 | https://codeium.com/windsurf |
| 发布日期 | 2024 |
| 定价 | Pro $15/月, Teams $30/月 |
| 目标用户 | 中高级开发者 |
| 底层模型 | Claude Sonnet, GPT-4o（混合路由） |

## 核心能力

1. Cascade：流式 Agent，逐步推理和执行
2. 深度代码库理解
3. 自动上下文管理

## 工作流

```
用户在 Windsurf IDE 中编码
  ├─ Cascade 激活 → 分析代码库上下文
  ├─ 流式推理 → 逐步生成执行计划
  ├─ 自动执行 → 文件编辑、终端命令
  ├─ 上下文持续更新 → 保持任务连贯性
  └─ 输出结果 → 用户确认
```

## 技术架构

- **基础**: VS Code fork
- **核心**: Cascade 引擎（流式 Agent，上下文感知）
- **模型**: 多模型混合路由
- **特色**: 自动上下文管理，无需手动 @ 引用

## 与同类对比

| 特性 | Windsurf | Cursor | Claude Code |
|------|----------|--------|-------------|
| 优势 | 流式体验自然、上下文自动管理 | Tab 补全、VS Code 生态 | 终端原生、MCP 生态 |
| 劣势 | 生态较小、插件少 | Agent 深度有限 | 无 GUI |

## 最佳实践

<!-- 待填写 -->

## 已知局限

<!-- 待填写 -->

## 使用心得

<!-- 个人使用体验、踩坑记录 -->

## 参考资料

- [Windsurf 官网](https://codeium.com/windsurf)
