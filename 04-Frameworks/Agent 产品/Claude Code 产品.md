---
title: Claude Code
aliases: [Claude Code, claude-code]
vendor: Anthropic
category: AI 编码 Agent
pricing: 按 token 计费（Pro $20/月, Max $100/$200/月）
url: https://docs.anthropic.com/en/docs/claude-code
tags:
  - framework
  - topic/coding
  - status/seed
created: 2026-06-12
updated: 2026-06-12
---

# Claude Code

> Anthropic 的终端原生 AI 编码 Agent。在终端中运行，直接读写文件、执行命令、管理 Git。CLAUDE.md 项目配置 + Subagent 架构 + MCP 工具生态。

## 基本信息

| 项目 | 详情 |
|------|------|
| 厂商 | Anthropic |
| 官网 | https://docs.anthropic.com/en/docs/claude-code |
| 发布日期 | 2025（公测） |
| 定价 | 按 token 计费；Pro $20/月, Max $100/$200/月 |
| 目标用户 | 专业开发者、工程团队 |
| 底层模型 | Claude Sonnet 4 / Claude Opus 4 |

## 核心能力

1. 终端原生，直接操作文件系统
2. CLAUDE.md 项目级指令配置
3. Subagent 并行编排
4. 200+ MCP Server 生态
5. Hooks 系统（生命周期钩子）

## 工作流

```
用户终端 → Claude Code CLI
  ├─ 读取 CLAUDE.md 项目配置
  ├─ 解析用户指令
  ├─ 规划任务 → 拆分子任务
  ├─ Subagent 并行执行（文件读写、命令执行、Git 操作）
  ├─ 代码审查 + 测试
  └─ 输出结果 / 提交 PR
```

## 技术架构

- **协议**: MCP（Model Context Protocol）连接外部工具
- **模型**: Claude Sonnet 4 / Opus 4，支持 extended thinking
- **扩展**: Hooks（PreToolUse / PostToolUse / Stop）、Subagent 编排
- **配置**: CLAUDE.md 分层配置（全局 → 项目 → 目录）

## 与同类对比

| 特性 | Claude Code | Cursor | Windsurf |
|------|------------|--------|----------|
| 优势 | 终端原生、Subagent 并行、MCP 生态 | IDE 集成、Tab 补全 | 流式 Cascade、上下文管理 |
| 劣势 | 无 GUI 编辑器、需终端操作 | Agent 能力较弱 | 生态较小 |

## 最佳实践

→ [[../Claude Code/CLAUDE.md 最佳实践|CLAUDE.md 最佳实践]]
→ [[../Claude Code/Agent 编排技巧|Agent 编排技巧]]

## 已知局限

- 纯终端交互，无图形化界面
- 大型重构时 context window 有限
- MCP Server 质量参差不齐

## 使用心得

<!-- 个人使用体验、踩坑记录 -->

## 参考资料

- [Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code)
