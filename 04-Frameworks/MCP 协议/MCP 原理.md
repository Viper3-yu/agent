---
title: MCP 协议（Model Context Protocol）
aliases:
  - MCP
  - Model Context Protocol
tags:
  - protocol
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-13
spec_url: https://modelcontextprotocol.io/specification
maintainer: Anthropic
transport: stdio / SSE / Streamable HTTP
---

# MCP 协议（Model Context Protocol）

> Anthropic 于 2024 年 11 月开源。为 LLM 提供标准化的工具/数据/提示词接口，被誉为"AI 时代的 USB-C"。

## 协议概览

| 项目 | 详情 |
|------|------|
| 全称 | Model Context Protocol (MCP) |
| 提出者 | Anthropic |
| 规范链接 | [MCP Specification](https://modelcontextprotocol.io/specification) |
| 传输层 | stdio / SSE / Streamable HTTP |
| 消息格式 | JSON-RPC 2.0 |
| 方向 | 垂直（模型→工具） |

## 设计动机

在 MCP 出现之前，每个 LLM 框架（LangChain / LlamaIndex / 自研）都要为每一个外部工具（数据库、API、文件系统）写一套适配器。**N × M 的适配困境**：

```
        数据库   GitHub   Slack   文件系统
LangChain   ✍️     ✍️      ✍️      ✍️
LlamaIndex  ✍️     ✍️      ✍️      ✍️
自研框架    ✍️     ✍️      ✍️      ✍️
```

MCP 把"工具接入"标准化为一套协议：

```
           MCP Server（按协议暴露能力）
                ↓
        MCP 协议（标准 JSON-RPC）
                ↓
        MCP Client（任意 LLM 应用）
```

这样框架只需实现一次 MCP Client，工具只需实现一次 MCP Server，**N × M → N + M**。

## 核心架构

```
┌──────────────────────────────────┐
│           MCP Host               │
│  （如 Claude Desktop / Cursor）   │
│                                  │
│  ┌──────────┐    ┌──────────┐   │
│  │ Client A │    │ Client B │   │
│  └────┬─────┘    └────┬─────┘   │
└───────┼───────────────┼─────────┘
        │ stdio/SSE/HTTP │
   ┌────▼─────┐    ┌─────▼────┐
   │ Server 1 │    │ Server 2 │
   │ (GitHub) │    │ (Files)  │
   └──────────┘    └──────────┘
```

- **Host**：承载 LLM 应用的进程（如 Claude Desktop、IDE 插件）
- **Client**：在 Host 内，与 Server 维持一对一连接
- **Server**：暴露 Resources / Tools / Prompts 三大原语

## 关键概念

### Resources（资源）

可被 LLM 读取的数据源——文件、数据库行、API 响应。类似 GET 端点。

```json
{
  "uri": "file:///docs/spec.md",
  "name": "产品规范",
  "mimeType": "text/markdown"
}
```

### Tools（工具）

可被 LLM 调用的函数，类似 OpenAI Function Calling 但标准化。

```json
{
  "name": "create_issue",
  "description": "在 GitHub 仓库创建 issue",
  "inputSchema": {
    "type": "object",
    "properties": {
      "repo": { "type": "string" },
      "title": { "type": "string" },
      "body": { "type": "string" }
    },
    "required": ["repo", "title"]
  }
}
```

### Prompts（提示词模板）

可复用的、带参数的用户提示词模板，由 Host 主动呈现给用户选择。

### Sampling（采样回拨）

Server 可以反过来请求 Host 的 LLM 完成生成——用于"Agent 调用 Agent"场景。少用，但开启了双向能力。

## 消息类型

MCP 在 JSON-RPC 2.0 之上定义了一组标准方法：

| 方法 | 方向 | 用途 |
|------|------|------|
| `initialize` | Client→Server | 握手，协商协议版本和能力 |
| `tools/list` | Client→Server | 枚举可用工具 |
| `tools/call` | Client→Server | 调用工具 |
| `resources/list` | Client→Server | 枚举可用资源 |
| `resources/read` | Client→Server | 读取资源 |
| `prompts/list` | Client→Server | 枚举提示词模板 |
| `prompts/get` | Client→Server | 获取并填充提示词 |
| `notifications/*` | Server→Client | 服务端主动通知（如资源更新） |
| `sampling/createMessage` | Server→Host | 反向调用 LLM |

## 交互流程

```
1. Client ──initialize──>     Server   （握手，交换能力）
2. Client <──capabilities─── Server
3. Client ──tools/list──>     Server   （发现工具）
4. Client <──tools────────── Server   （工具清单）
5. LLM 决定调用工具
6. Client ──tools/call──>    Server   （执行）
7. Client <──result───────── Server   （返回结果）
8. 结果送回 LLM 继续推理
```

## 安全考量

- **授权模型**：用户在 Host 端**显式同意**每个 Server 的能力范围
- **作用域限制**：Tool 调用可声明 `destructive: true` 等元数据，Host 可二次确认
- **沙箱**：stdio 模式下 Server 在 Host 子进程中运行，权限受用户账号限制
- **审计日志**：所有 tool call 可记录到 Host 日志
- **OAuth 2.1**：Streamable HTTP 传输下 Server 可用 OAuth 2.1 鉴权

## 与其他协议的关系

| 协议 | 方向 | 解决的问题 | 与 MCP 关系 |
|------|------|----------|------------|
| MCP | 模型↔工具 | 工具调用标准化 | — |
| [[../A2A 协议/A2A 原理\|A2A]] | Agent↔Agent | Agent 间协作 | 互补：MCP 连工具，A2A 连 Agent |
| [[../AG-UI 协议/AG-UI 原理\|AG-UI]] | Agent↔UI | 前端流式交互 | 互补：MCP 是后端，AG-UI 是前端 |
| OpenAI Function Calling | 模型↔工具 | 单厂商工具调用 | MCP 的前身思想；MCP 跨厂商、跨框架 |
| LangChain Tools | 模型↔工具 | 框架内工具抽象 | LangChain 现已实现 MCP Client |

## 代码示例

```python
# 服务端：定义一个简单 MCP Server
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("demo-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="echo",
            description="回显输入文本",
            inputSchema={
                "type": "object",
                "properties": {"text": {"type": "string"}},
                "required": ["text"],
            },
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "echo":
        return [TextContent(type="text", text=arguments["text"])]
    raise ValueError(f"Unknown tool: {name}")

# 客户端：连接 Server 并调用
from mcp.client import stdio_client

async with stdio_client(app) as (read, write):
    tools = await app.list_tools()  # 发现工具
    result = await app.call_tool("echo", {"text": "hello MCP"})
    print(result)
```

## 版本演进

| 版本 | 日期 | 亮点 |
|------|------|------|
| 2024-11-05 | 首发 | Anthropic 开源，stdio + JSON-RPC |
| 2025-03 | SSE | 增加 SSE 远程传输 |
| 2025-06 | 资源+授权 | Resources、Roots、OAuth 鉴权 |
| 2025-11 | Streamable HTTP | 替换 SSE，支持双向流式 |
| 2026 | 1.0 候选 | 协议稳定，生态规模化 |

## 参考资料

- [MCP 官方规范](https://modelcontextprotocol.io/specification)
- [MCP GitHub](https://github.com/modelcontextprotocol)
- [Anthropic 公告](https://www.anthropic.com/news/model-context-protocol)
- 详见 [[MCP Server 开发]] 动手教程

相关概念：[[../../01-Concepts/Agent 架构/ReAct 架构|ReAct 架构]]