---
title: A2A Agent Skills
aliases: [A2A AgentSkill, AgentCard Skills]
tags:
  - framework
  - topic/agent
  - status/mature
created: 2026-06-12
updated: 2026-06-12
language: JSON
version: "2025"
repo: https://github.com/a2aproject/A2A
license: Apache 2.0
---

# A2A Agent Skills

> A2A 协议中 Agent Card 的核心组成部分。每个 A2A Agent 通过 Agent Card 声明自己的 Skills，让其他 Agent 能发现并理解自己的能力。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://github.com/a2aproject/A2A |
| GitHub | https://github.com/a2aproject/A2A |
| 语言 | JSON（协议格式） |
| 最新版本 | 2025（初始发布） |
| 许可证 | Apache 2.0 |

## 核心架构

A2A Agent Skills 是 Agent Card 的子结构，用于声明 Agent 具备的能力。其他 Agent 通过读取 Agent Card 中的 skills 数组来发现和匹配可用能力。

### 在 A2A 协议中的位置

```
Agent Card
├── name, description, url, version
├── skills[]          ← 这里
│   ├── id
│   ├── name
│   ├── description
│   ├── tags
│   ├── examples
│   ├── inputModes
│   └── outputModes
├── capabilities
├── security
└── interfaces[]
```

## 快速上手

### 最小示例

```json
{
  "skills": [
    {
      "id": "summarize",
      "name": "Document Summarizer",
      "description": "Summarizes long documents into key points",
      "tags": ["text", "summarization"],
      "examples": ["Summarize this research paper"],
      "inputModes": ["text/plain", "application/pdf"],
      "outputModes": ["text/plain"]
    }
  ]
}
```

## 关键概念

### AgentSkill 对象字段

| 字段 | 说明 |
|------|------|
| `id` | Skill 唯一标识 |
| `name` | 人类可读名称 |
| `description` | 能力描述（供其他 Agent 理解匹配） |
| `tags` | 分类标签 |
| `examples` | 输入示例（帮助 Agent 匹配任务） |
| `inputModes` | 支持的输入格式（text/plain, application/json 等） |
| `outputModes` | 支持的输出格式 |

### Agent 间能力发现机制

Agent 通过 HTTP 请求获取目标 Agent 的 Agent Card，解析 skills 数组来判断是否具备所需能力，然后发起任务请求。

## 典型使用场景

1. **跨组织 Agent 协作**: 不同团队的 Agent 通过 Agent Card 互相发现能力
2. **Agent 市场**: Agent 注册中心聚合各 Agent 的 Skills 供用户搜索
3. **动态任务路由**: 根据 skills 匹配将任务分发给最合适的 Agent

## 与同类对比

| 体系 | 用途 | 详见 |
|------|------|------|
| **A2A AgentSkill** | Agent 间能力发现（本文） | 即本文 |
| Agent Skills (SKILL.md) | 打包知识给 Agent 加载 | [[./Agent Skills 规范]] |
| AGENTS.md | 仓库级 Agent 可发现性 | [[./AGENTS.md 规范]] |
| MCP Tools | 模型调用工具的能力声明 | [[../MCP 协议/MCP 原理|MCP]] |

## 已知局限

- A2A 协议尚处于早期阶段，工具链支持有限
- Skills 匹配依赖文本描述，缺乏语义匹配机制
- 与 MCP Tools 的能力声明存在重叠

## 参考资料

- [A2A 规范](https://github.com/a2aproject/A2A/blob/main/docs/specification.md)
- [Google A2A 公告](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
- [[../A2A 协议/A2A 原理|A2A 协议]]
