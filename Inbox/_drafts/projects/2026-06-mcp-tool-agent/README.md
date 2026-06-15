---
title: 用 MCP 协议搭一个工具调用 Agent
aliases: [MCP Tool Agent, MCP-Agent-Demo]
tags:
  - project
  - topic/agent
  - status/draft
created: 2026-06-15
updated: 2026-06-15
status: 进行中
---

# 用 MCP 协议搭一个工具调用 Agent

> 从零搭一个**最小但完整**的 MCP-based Agent：自己写 MCP Server（暴露 3 个工具），自己写 MCP Client（用 Anthropic / Claude API 调用 LLM 做工具选择），跑通 "用户问问题 → Agent 选工具 → Server 返回 → Agent 综合回答" 的完整循环。

## 项目目标

- **核心目标**：跑通 MCP 协议的 end-to-end 最小闭环
- **次要目标**：理解 MCP 与传统 function calling 的差异
- **远期目标**：把 vault 里的 Clippings / Resources 接入 MCP Server，让 Agent 能直接读 vault 内容

## 背景与动机

vault 里 [[../../../04-Frameworks/MCP 协议/MCP 原理|MCP 原理]] 和 [[../../../04-Frameworks/MCP 协议/MCP Server 开发|MCP Server 开发]] 笔记都写好了，但**没跑过**就是纸上谈兵。Anthropic 把 MCP 推成 Agent 工具调用的标准，**不亲手搭一次搞不懂"为什么这样设计"**。

跟 [[../2026-06-repro-building-effective-agents/README|复现 Building effective agents]] 互补：那个是范式广度，这个是协议深度。

## 技术方案

### 架构

```
┌─────────────────┐  JSON-RPC over stdio/SSE  ┌──────────────────┐
│   MCP Client    │◀──────────────────────────▶│   MCP Server     │
│   (Agent 端)    │                            │   (工具提供端)   │
│                 │                            │                  │
│ - LLM 调用      │                            │ - 工具注册       │
│ - 工具选择      │                            │ - 资源暴露       │
│ - 上下文管理    │                            │ - 权限控制       │
└─────────────────┘                            └──────────────────┘
        │                                              │
        │ Claude API                                   │ 本地文件 / 外部 API
        ▼                                              ▼
   Anthropic /                              vault Clippings / 真实 API
   OpenAI
```

### MCP Server（3 个工具）

| 工具名 | 用途 | 实现 |
|--------|------|------|
| `search_vault` | 在 vault Clippings 里搜主题 | 简单 BM25（起步），可升级到 Embedding |
| `read_note` | 读 vault 笔记全文 | 文件读 + frontmatter parse |
| `list_topics` | 列出最近 N 天剪藏的标签 | 扫 frontmatter `tags` 字段 |

### MCP Client（Agent 端）

- **LLM**：`claude-sonnet-4-6`（Anthropic 官方 SDK）
- **协议**：`mcp` Python SDK（`pip install mcp`）
- **工具选择**：直接用 Anthropic 的 tool use（Claude 原生支持）
- **循环**：用户问题 → 拼 prompt + 工具列表 → Claude 决定调哪个工具 → 调 → 把结果拼回 prompt → 循环直到 Claude 说 done → 输出

### 跑通的"演示场景"

> 用户：上周关于 Agent 安全的文章有哪几篇？
>
> → Agent 调 `list_topics("agent 安全", days=7)`
> → Server 返回 3 个笔记链接
> → Agent 调 `read_note` 读每个笔记的摘要
> → Agent 综合输出"上周共 3 篇关于 Agent 安全的剪藏..."

## 里程碑

- [ ] **M1（1 周）** — MCP Server 搭起来，3 个工具 + 单元测试
- [ ] **M2（2 周）** — MCP Client 调通，跑通"用户问 → 单次工具调用 → 回答"循环
- [ ] **M3（3 周）** — 多轮工具调用（一次会话里调多次）+ 错误处理
- [ ] **M4（4 周）** — 接入真实 vault Clippings，跑通演示场景
- [ ] **M5（5 周）** — 写 vault 复盘笔记，含 trace 截图 + MCP vs function calling 对比

## 已读资料

- [[../../../04-Frameworks/MCP 协议/MCP 原理|MCP 原理]] — 协议基础
- [[../../../04-Frameworks/MCP 协议/MCP Server 开发|MCP Server 开发]] — 实现细节
- [MCP 官方 Python SDK 文档](https://modelcontextprotocol.io/) — API 参考
- [Anthropic Tool Use 文档](https://docs.anthropic.com/en/docs/tool-use) — 工具调用模式

## 待读资料

- MCP 官方 Quickstart（跑个例子）
- [Claude Agent SDK 如何用 MCP](https://docs.claude.com/en/api/agent-sdk) — 看工业界怎么用
- MCP Inspector（调试工具）

## 进展日志

### 2026-06-15
- 项目立项：从方案 §5.1 采纳，进入 Active
- 创建项目文件夹骨架
- 待办：M1 启动前先把 MCP 原理笔记重新读一遍

## 风险与回避

| 风险 | 应对 |
|------|------|
| MCP 协议还在演进 | 锁 SDK 版本，breaking change 时再升级 |
| Anthropic API 国内访问 | 优先用 OpenAI 兼容的（vLLM 自部署也行） |
| vault 文件被 MCP 误改 | Server 只暴露 read-only 工具，写入走单独审批流 |
| 调试 trace 太长 | 用 LangSmith / MCP Inspector 单独面板 |

## 产出物（最终交付）

1. **1 个 MCP Server**（3 个工具 + 测试）
2. **1 个 MCP Client**（支持多轮 + 错误处理）
3. **1 篇 vault 复盘笔记**（含 trace + 反思 + MCP vs function calling 对比表）
4. **1 个 README 演示脚本**（一键跑通演示场景）

## 相关资源

- → [[../../../00-MOC/项目看板|项目看板 MOC]]
- → [[../../../00-MOC/框架与工具地图|框架与工具地图]]
- → [[../../../04-Frameworks/MCP 协议/MCP 原理|MCP 原理]]
- → [[../2026-06-repro-building-effective-agents/README|Building effective agents 复现]]（互补项目）
- → [[../README|Active 看板]]
