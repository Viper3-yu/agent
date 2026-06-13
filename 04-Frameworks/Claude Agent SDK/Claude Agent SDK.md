---
title: Claude Agent SDK
aliases: [Claude Agent SDK, Claude Code SDK, Anthropic Agent SDK]
tags:
  - framework
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
language: Python, TypeScript
version: "2026-04"
repo: "N/A（闭源 SDK）"
license: 商业许可
---

# Claude Agent SDK

> Anthropic 的官方 Agent 开发框架。深度集成 Claude Code 的 subagent 架构，原生 MCP 支持（生态最深），CLAUDE.md 上下文注入。2026 年 4 月推出 Claude Managed Agents 托管平台。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://docs.anthropic.com |
| GitHub | N/A（闭源 SDK） |
| 语言 | Python, TypeScript |
| 最新版本 | 2026-04 |
| 许可证 | 商业许可 |

## 核心架构

- `query()` 原语：核心 Agent 执行
- Subagent：独立上下文的子 Agent
- Hook 生命周期：pre/post 工具调用拦截
- Schema 验证：结构化输出保障
- 流式处理：实时流式响应

## 快速上手

### 安装

```bash
pip install claude-agent-sdk
```

### 最小示例

```python
from claude_agent_sdk import Agent

agent = Agent(model="claude-sonnet-4-20250514")
result = agent.query("帮我分析这个代码库的架构")
print(result)
```

## 关键概念

### 核心特性

| 特性 | 说明 |
|------|------|
| 语言 | Python, TypeScript |
| 多 Agent | Subagent 隔离 + Hook 生命周期 |
| MCP 支持 | 原生（生态最深，200+ server 单行配置） |
| A2A 支持 | 暂不支持 |
| 模型范围 | 仅 Claude 系列 |
| 部署 | 自托管或 API |

### Subagent 架构

每个 Subagent 拥有独立上下文，通过 Hook 生命周期实现工具调用前后的拦截和处理。

### MCP 原生集成

200+ MCP server 单行配置即可接入，生态覆盖最深。

## 典型使用场景

1. **编码 Agent**: Claude Code 本身就是最佳案例
2. **深度 MCP 集成**: 需要连接多种外部工具和数据源
3. **长链路多步骤工作流**: 复杂的开发工作流编排

## 与同类对比

| 特性 | Claude Agent SDK | OpenAI Agents SDK | Google ADK |
|------|-----------------|-------------------|------------|
| 模型绑定 | 仅 Claude | 仅 OpenAI | Gemini 优化，模型无关 |
| MCP 支持 | 原生（最深） | 已采纳 | 适配器 |
| A2A 支持 | 暂不支持 | 暂不支持 | 原生（最强） |
| 多语言 | Python, TS | Python, TS | Python, TS, Java, Go, Kotlin |

→ 详见 [[../OpenAI Agents SDK/OpenAI Agents SDK|OpenAI Agents SDK]]
→ 详见 [[../Google ADK/Google ADK|Google ADK]]

## 已知局限

- 仅支持 Claude 系列模型，无法切换其他厂商
- 暂不支持 A2A 跨厂商 Agent 互操作
- SDK 闭源，定制能力有限

## 参考资料

- [Morph 框架对比](https://www.morphllm.com/ai-agent-framework)
- [TURION.AI 三方对比](https://turion.ai/blog/google-adk-vs-openai-claude-agent-sdk-2026/)
- [AgentMarketCap 分析](https://agentmarketcap.ai/blog/2026/04/11/openai-responses-api-vs-google-adk-vs-anthropic-agent-sdk-2026)
