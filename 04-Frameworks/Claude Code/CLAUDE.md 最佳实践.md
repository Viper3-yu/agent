---
title: CLAUDE.md 最佳实践
aliases:
  - CLAUDE.md
tags:
  - framework
  - status/seed
created: 2026-06-12
updated: 2026-06-12
language: Markdown
version: "持续更新"
repo: "N/A（Claude Code 内置）"
license: 商业许可
---

# CLAUDE.md 最佳实践

> Claude Code 的项目配置文件最佳实践

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://docs.anthropic.com/claude-code |
| GitHub | N/A（Claude Code 内置） |
| 语言 | Markdown |
| 最新版本 | 持续更新 |
| 许可证 | 商业许可 |

## 核心架构

CLAUDE.md 是 Claude Code 的项目级配置文件，用于定义项目规范、编码风格和工作流程。支持层级结构：全局（~/.claude/CLAUDE.md）、项目根目录、子目录。

## 快速上手

### 创建 CLAUDE.md

在项目根目录创建 `CLAUDE.md` 文件，定义项目规范和工作流程。

### 最小示例

```markdown
# CLAUDE.md

## 技术栈
- TypeScript + React
- PostgreSQL
- Vercel 部署

## 命令
- `npm run build` - 构建
- `npm test` - 测试
- `npm run lint` - 检查

## 规范
- 使用 conventional commits
- 函数不超过 50 行
- 文件不超过 800 行
```

## 关键概念

### 层级结构

| 层级 | 路径 | 作用域 |
|------|------|--------|
| 全局 | `~/.claude/CLAUDE.md` | 所有项目 |
| 项目 | `./CLAUDE.md` | 当前项目 |
| 子目录 | `./src/CLAUDE.md` | 特定模块 |

### 常用配置项

- 定义项目技术栈和依赖
- 指定编码规范和命名约定
- 配置测试和构建命令
- 设定安全和审查规则

## 典型使用场景

1. **项目规范声明**: 告知 Claude Code 项目的构建、测试、部署流程
2. **编码约定传达**: 指定命名规范、代码风格、分支策略
3. **Agent 协作规则**: 定义多 Agent 场景下的行为规范

## 与同类对比

| 维度 | CLAUDE.md | AGENTS.md |
|------|-----------|-----------|
| 范围 | Claude Code 专用 | 通用跨框架 |
| 格式 | Markdown | Markdown |
| 支持 | Claude Code | 多种 Agent |

## 已知局限

- 仅 Claude Code 支持
- 缺乏 schema 验证
- 配置冲突时优先级规则不直观

## 参考资料

- [Claude Code 文档](https://docs.anthropic.com/claude-code)
- [[./Agent 编排技巧|Agent 编排技巧]]
