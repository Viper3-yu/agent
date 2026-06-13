---
title: MCP Server 开发
aliases:
  - MCP Server
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

# MCP Server 开发

> 动手教程：从零开发一个生产级 MCP Server。

## 协议概览

| 项目 | 详情 |
|------|------|
| 全称 | Model Context Protocol (MCP) Server |
| 提出者 | Anthropic |
| 规范链接 | [MCP Specification](https://modelcontextprotocol.io/specification) |
| 传输层 | stdio / SSE / Streamable HTTP |
| 消息格式 | JSON-RPC 2.0 |
| 方向 | 垂直（模型→工具） |

> 协议原理见 [[MCP 原理]]，本篇专注**实现**。

## 开发环境

### 官方 SDK

| 语言 | 包 | 安装 | 仓库 |
|------|----|------|------|
| Python | `mcp` | `pip install mcp` | [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk) |
| TypeScript | `@modelcontextprotocol/sdk` | `npm install @modelcontextprotocol/sdk` | 同组织 TS SDK |
| Go | `mcp-go` | 社区维护 | [mark3labs/mcp-go](https://github.com/mark3labs/mcp-go) |
| Rust | `rmcp` | 官方 | [modelcontextprotocol/rust-sdk](https://github.com/modelcontextprotocol/rust-sdk) |
| Kotlin | — | JetBrains 官方 | GitHub modelcontextprotocol 组织 |
| Java | — | Spring AI 集成 | 社区 |

### 配套工具

- **MCP Inspector**：官方调试 GUI，支持浏览 Server 能力并手动调用
  ```bash
  npx @modelcontextprotocol/inspector python my_server.py
  ```
- **Claude Desktop**：本地运行的 MCP Host，可直接加载 stdio Server
- **Cursor / Windsurf**：IDE 类 Host，远程 Server 需 Streamable HTTP

## 三大原语的代码模式

### 1. Tool（最常用）

```python
from mcp.server import Server
from mcp.types import Tool, TextContent
import httpx

app = Server("github-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="search_repos",
            description="按关键词搜索 GitHub 仓库",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "搜索关键词"},
                    "limit": {"type": "integer", "default": 10, "minimum": 1, "maximum": 100},
                },
                "required": ["query"],
            },
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "search_repos":
        async with httpx.AsyncClient() as client:
            r = await client.get(
                "https://api.github.com/search/repositories",
                params={"q": arguments["query"], "per_page": arguments.get("limit", 10)},
            )
            data = r.json()
            items = [
                f"{repo['full_name']} (⭐ {repo['stargazers_count']}): {repo['description']}"
                for repo in data["items"]
            ]
            return [TextContent(type="text", text="\n".join(items))]
    raise ValueError(f"Unknown tool: {name}")
```

### 2. Resource（数据源）

```python
@app.list_resources()
async def list_resources() -> list[Resource]:
    return [
        Resource(
            uri="file:///workspace/README.md",
            name="项目 README",
            mimeType="text/markdown",
        )
    ]

@app.read_resource()
async def read_resource(uri: str) -> str:
    if uri.startswith("file://"):
        path = uri[7:]  # 去掉 file:// 前缀
        with open(path, encoding="utf-8") as f:
            return f.read()
    raise ValueError(f"Unsupported URI scheme: {uri}")
```

### 3. Prompt（带参数模板）

```python
@app.list_prompts()
async def list_prompts() -> list[Prompt]:
    return [
        Prompt(
            name="code_review",
            description="生成代码审查清单",
            arguments=[
                {"name": "language", "description": "编程语言", "required": True},
                {"name": "focus", "description": "重点关注点", "required": False},
            ],
        )
    ]

@app.get_prompt()
async def get_prompt(name: str, arguments: dict) -> PromptMessage:
    if name == "code_review":
        lang = arguments["language"]
        focus = arguments.get("focus", "通用质量")
        return PromptMessage(
            role="user",
            content=f"请以 {lang} 专家身份审查以下代码，重点关注：{focus}。"
        )
    raise ValueError(f"Unknown prompt: {name}")
```

## 传输选择

### stdio（最简单）

```python
from mcp.server.stdio import stdio_server

if __name__ == "__main__":
    asyncio.run(stdio_server(app))
```

适用于：
- 本地 CLI 工具
- Claude Desktop 配置的 Server
- 同一台机器的 Host

### Streamable HTTP（远程）

```python
from mcp.server.transport import streamable_http_server

if __name__ == "__main__":
    streamable_http_server(app, host="0.0.0.0", port=8080)
```

适用于：
- 远程 Server（团队共享）
- 多用户访问
- 部署到云端

## 调试技巧

```bash
# 用 Inspector 调试本地 Server
npx @modelcontextprotocol/inspector python -m my_server

# 用 Inspector 调试远程 Server
npx @modelcontextprotocol/inspector --url http://localhost:8080

# 启用详细日志
export MCP_LOG_LEVEL=DEBUG
python my_server.py 2> server.log
```

## Claude Desktop 配置

`~/Library/Application Support/Claude/claude_desktop_config.json`（macOS）：

```json
{
  "mcpServers": {
    "github": {
      "command": "python",
      "args": ["/path/to/github_server.py"],
      "env": { "GITHUB_TOKEN": "ghp_..." }
    }
  }
}
```

## 最佳实践

### 必做

- **明确的工具描述**：description 是 LLM 决定何时调用的唯一依据
- **严格的 inputSchema**：用 JSON Schema 的 `required` / `minimum` / `enum` 约束
- **错误返回结构化**：Tool 抛错时返回包含错误码的 TextContent，不要让进程崩溃
- **幂等设计**：Tool 多次调用不应产生副作用叠加
- **超时控制**：每个 Tool call 设置超时（如 30s），避免阻塞 LLM

### 不要

- 不要在 Tool 里做 LLM 调用（Sampling 是更好选择）
- 不要假设调用方是同一个用户（Streamable HTTP 模式下要鉴权）
- 不要把 secrets 硬编码到 Server 代码（用环境变量）

## 安全清单

- [ ] Tool 是否声明了 `destructive` 元数据（删除、修改类）
- [ ] 是否启用了 Host 端的用户确认
- [ ] 远程 Server 是否配置了 OAuth 2.1 鉴权
- [ ] 日志是否脱敏（不打印用户输入的敏感字段）
- [ ] 依赖是否定期更新（防止供应链攻击）

## 官方 Server 索引

| Server | 用途 | 传输 |
|--------|------|------|
| `@modelcontextprotocol/server-filesystem` | 文件系统读写 | stdio |
| `@modelcontextprotocol/server-git` | Git 操作 | stdio |
| `@modelcontextprotocol/server-github` | GitHub API | stdio |
| `@modelcontextprotocol/server-postgres` | PostgreSQL | stdio |
| `@modelcontextprotocol/server-slack` | Slack | stdio |
| `@modelcontextprotocol/server-puppeteer` | 浏览器自动化 | stdio |

完整列表：https://github.com/modelcontextprotocol/servers

## 参考资料

- [MCP 规范](https://modelcontextprotocol.io/specification)
- [Python SDK 文档](https://github.com/modelcontextprotocol/python-sdk)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- 协议原理：[[MCP 原理]]