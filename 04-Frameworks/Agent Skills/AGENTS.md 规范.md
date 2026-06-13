---
title: AGENTS.md 规范
aliases: [AGENTS.md, Agent Discoverability]
tags:
  - framework
  - topic/agent
  - status/reviewed
  - meta/stale
created: 2026-06-12
updated: 2026-06-12
language: Markdown
version: "草案"
repo: "N/A（社区规范）"
license: 开放标准
---


> ⚠️ 联网核查未通过（沙箱限制）；事实/版本可能过时，请审阅。— 2026-06-13
# AGENTS.md 规范

> 仓库级别的 Agent 可发现性规范，类似 `.editorconfig` 之于编辑器。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | N/A（社区规范） |
| GitHub | N/A |
| 语言 | Markdown |
| 最新版本 | 草案阶段 |
| 许可证 | 开放标准 |

## 核心架构

AGENTS.md 是放在代码仓库根目录的声明文件，告诉 AI Agent：
- 这个项目是什么
- 可用的工具和能力
- 项目约定和限制
- 推荐的工作方式

## 快速上手

### 创建 AGENTS.md

在仓库根目录创建 `AGENTS.md` 文件，声明项目信息和 Agent 工作规范。

### 最小示例

```markdown
# AGENTS.md

## 项目概述
这是一个 TypeScript monorepo，使用 pnpm 管理。

## 构建命令
- `npm run build` - 构建项目
- `npm test` - 运行测试
- `npm run lint` - 代码检查

## 约定
- 使用 conventional commits
- PR 需要至少 1 个 review
```

## 关键概念

### 与 Claude Code 的关系

Claude Code 的 `CLAUDE.md` 是 AGENTS.md 概念的先行实践：
- `CLAUDE.md`：Claude Code 专用的项目配置
- `AGENTS.md`：通用的、跨框架的 Agent 项目配置

### 层级结构

支持全局、项目级、子目录级的多层配置，优先级从高到低。

## 典型使用场景

1. **项目规范声明**: 告知 Agent 项目的构建、测试、部署流程
2. **编码约定传达**: 指定命名规范、代码风格、分支策略
3. **安全边界定义**: 声明哪些文件/目录不应被修改

## 与同类对比

| 维度 | AGENTS.md | CLAUDE.md | SKILL.md |
|------|-----------|-----------|----------|
| 范围 | 仓库级 | 项目级 | 能力级 |
| 对象 | 所有 Agent | Claude Code | 单个 Skill |
| 内容 | 项目概述+约定 | 详细配置+规则 | 专业知识+工作流 |

## 已知局限

- 规范尚为草案，缺乏正式版本号
- 不同 Agent 对 AGENTS.md 的支持程度不一
- 缺乏 schema 验证机制

## 参考资料

- [[../../Claude Code/CLAUDE.md 最佳实践|Claude Code CLAUDE.md]]
- [[../A2A 协议/A2A 原理|A2A Agent Cards]]
