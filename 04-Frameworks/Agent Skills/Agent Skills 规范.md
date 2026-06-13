---
title: Agent Skills 规范
aliases: [Agent Skills, SKILL.md, agentskills]
tags:
  - framework
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
language: Markdown/YAML
version: "2025-12"
repo: https://github.com/agentskills/agentskills
license: 开放标准
---

# Agent Skills 规范

> Anthropic 开发并开源的 Agent 能力扩展标准。用一个包含 `SKILL.md` 的文件夹打包专业知识和工作流，让任何兼容的 Agent 加载使用。2025 年 12 月发布为开放标准。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://agentskills.io |
| GitHub | https://github.com/agentskills/agentskills |
| 语言 | Markdown/YAML |
| 最新版本 | 2025-12（初始发布） |
| 许可证 | 开放标准 |

## 核心架构

Agent 很强大，但往往缺少执行真实任务所需的上下文。Skills 解决这个问题：把**专业知识和特定工作流**打包成可移植、可版本控制的文件夹，Agent 按需加载。

### Skill 目录结构

```
skill-name/
├── SKILL.md          # 必需：元数据 + 指令
├── scripts/          # 可选：可执行代码
├── references/       # 可选：参考文档
├── assets/           # 可选：模板、资源
└── ...               # 任意附加文件
```

## 快速上手

### 创建一个 Skill

在任意目录下创建 `SKILL.md`，填入 frontmatter 和指令正文即可。

### 最小示例

```yaml
---
name: pdf-extractor
description: >
  Extracts text and tables from PDF files, fills PDF forms,
  and converts PDF pages to images. Use when working with
  PDF documents or form processing.
compatibility: Requires Python 3.10+, poppler-utils
allowed-tools: "bash python"
---
```

Frontmatter 之后的 Markdown 正文就是**指令内容**，告诉 Agent 如何执行任务。

## 关键概念

### SKILL.md Frontmatter 字段

| 字段 | 必需 | 限制 |
|------|:---:|------|
| `name` | ✅ | 最多 64 字符，小写字母+数字+连字符 |
| `description` | ✅ | 最多 1024 字符，描述功能和使用时机 |
| `license` | ❌ | 许可证名称 |
| `compatibility` | ❌ | 最多 500 字符，环境要求 |
| `metadata` | ❌ | 任意键值对 |
| `allowed-tools` | ❌ | 预授权工具列表（实验性） |

### 渐进式加载（Progressive Disclosure）

这是 Agent Skills 最重要的设计：

| 阶段 | 加载内容 | Token 消耗 |
|------|---------|-----------|
| ① Discovery | 仅 name + description | ~100 tokens |
| ② Activation | 完整 SKILL.md 正文 | < 5000 tokens |
| ③ Execution | scripts/references/assets | 按需加载 |

Agent 启动时只扫描所有 Skill 的 name 和 description，任务匹配时才加载完整指令。可以同时携带大量 Skills，只消耗极少上下文。

### 设计原则

1. **SKILL.md 控制在 500 行以内**，详细内容拆分到 references/
2. **description 要精准**，包含关键词帮助 Agent 匹配任务
3. **scripts 应可独立执行**，包含必要的错误处理
4. **跨产品复用**，一次构建，任何兼容 Agent 可用

## 典型使用场景

1. **专业知识打包**: 将 PDF 处理、数据分析等专业技能打包为 Skill
2. **工作流标准化**: 将团队最佳实践编码为可复用的 Skill
3. **跨 Agent 复用**: 一次构建，Claude Code、Cursor 等兼容 Agent 均可加载

## 与同类对比

| 维度 | Agent Skills (SKILL.md) | A2A AgentSkill |
|------|------------------------|----------------|
| 发起者 | Anthropic | Google |
| 定位 | 打包专业知识给 Agent | Agent 间能力声明 |
| 格式 | 文件夹 + SKILL.md | JSON 对象 |
| 加载方式 | 渐进式（本地文件） | Agent Card（HTTP 发现） |
| 关系 | Agent 内部能力扩展 | Agent 间通信协议 |

## 已知局限

- 标准尚处于早期阶段，生态兼容性有待扩展
- `allowed-tools` 字段仍为实验性
- 缺乏版本管理和依赖解析机制

## 参考资料

- [Agent Skills 官网](https://agentskills.io)
- [GitHub 规范](https://github.com/agentskills/agentskills)
- [[./A2A Agent Skills|A2A Agent Skills]]
- [[./AGENTS.md 规范|AGENTS.md 规范]]
