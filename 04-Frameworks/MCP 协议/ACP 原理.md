---
title: ACP 协议（Agent Communication Protocol）
aliases: [ACP, Agent Communication Protocol]
tags:
  - protocol
  - topic/agent
  - status/archived
created: 2026-06-12
updated: 2026-06-13
spec_url: https://github.com/i-am-bee/agent-communication-protocol
maintainer: IBM Research / BeeAI（已移交 Linux Foundation）
transport: HTTP (REST)
---

# ACP 协议（Agent Communication Protocol）

> ⚠️ **ACP 已于 2025 年 8 月并入 [[../A2A 协议/A2A 原理|A2A]]**。本页作为**演进史档案**保留，新项目应直接使用 A2A。

IBM Research / BeeAI 于 2025 年 3 月发布的轻量级 Agent 通信协议，主打"REST 原生 + 多模态消息"。其核心创新已被 A2A v0.3 吸收。

## 协议概览

| 项目 | 详情 |
|------|------|
| 全称 | Agent Communication Protocol (ACP) |
| 提出者 | IBM Research / BeeAI |
| 规范链接 | [ACP GitHub](https://github.com/i-am-bee/agent-communication-protocol) |
| 传输层 | HTTP (REST) |
| 消息格式 | JSON（多模态 Part 模型） |
| 方向 | 水平（Agent↔Agent） |
| 状态 | 已归档，并入 A2A |

## 设计动机

ACP 在 A2A 之后发布（2025 年 4 月），它试图解决 A2A 早期版本的几个痛点：

- **REST 原生**：无需专用 SDK，标准 HTTP 即可测试
- **多模态消息**：原生支持文本、图像、文件等多 Part 消息
- **异步流式**：Server-Sent Events 支持长时间任务
- **可观测性**：内置 OpenTelemetry 追踪和监控

## 核心架构

```
┌──────────────┐    HTTP/JSON     ┌──────────────┐
│              │ ───────────────> │              │
│  ACP Client  │                  │  ACP Server  │
│  (Agent A)   │ <─────────────── │  (Agent B)   │
│              │   SSE 流式响应    │              │
└──────────────┘                  └──────────────┘

所有端点遵循 REST 风格：
  POST   /agents/{id}/runs        # 启动一次 Agent 运行
  GET    /agents/{id}/runs/{run_id} # 查询运行状态
  GET    /agents/{id}/runs/{run_id}/stream  # SSE 流式输出
  POST   /agents/{id}/runs/{run_id}/cancel  # 取消运行
```

## 关键概念

### Agent Manifest

类似 A2A 的 Agent Card，发布在 `/.well-known/agent.json`：

```json
{
  "name": "research-agent",
  "version": "0.3.1",
  "capabilities": ["web_search", "summarize"],
  "input_modes": ["text", "image"],
  "output_modes": ["text"]
}
```

### 多模态 Part

ACP 的核心创新——所有消息内容都用 Part 数组表示：

```json
{
  "role": "user",
  "parts": [
    { "type": "text", "text": "请分析这张图" },
    { "type": "image", "url": "https://..." },
    { "type": "file", "name": "data.csv", "data": "base64..." }
  ]
}
```

这种统一 Part 模型后来被 A2A v0.3+ 采纳为标准。

### Run / Step 状态机

```
queued → running → completed
                  → failed
                  → canceled
                  → awaiting_input (人类反馈)
```

## 消息类型

| 端点 | 方法 | 用途 |
|------|------|------|
| `/agents` | GET | 列出可用 Agent |
| `/agents/{id}/runs` | POST | 启动运行（含多模态 parts） |
| `/agents/{id}/runs/{run_id}` | GET | 查询运行状态 |
| `/agents/{id}/runs/{run_id}/stream` | GET (SSE) | 流式获取步骤输出 |
| `/agents/{id}/runs/{run_id}/cancel` | POST | 取消运行 |
| `/.well-known/agent.json` | GET | Agent 能力声明 |

## 交互流程

```
1. Client → GET /.well-known/agent.json   （发现 Agent 能力）
2. Client → POST /agents/research-agent/runs  （启动，含多模态输入）
3. Server → 返回 run_id
4. Client → GET /agents/{id}/runs/{run_id}/stream （SSE 订阅步骤）
5. Server → 持续推送 step 事件
6. Server → 最终事件 status=completed
7. Client 读取最终 parts 数组作为结果
```

## 安全考量

- **mTLS**：跨组织调用应启用双向 TLS
- **Token-based 鉴权**：Bearer Token 在 Authorization Header
- **速率限制**：每 Agent 端点应有 QPS 上限
- **审计日志**：所有 Run 记录保留 90 天
- **数据驻留**：多模态 Part 中的 PII 应在传输前脱敏

## 与其他协议的关系

### 与 A2A 的融合（2025 年 8 月）

| ACP 特性 | 在 A2A 中的位置 |
|---------|----------------|
| REST 端点 | A2A 的 HTTP+JSON 绑定 |
| 多模态 Part | A2A 的 Message/Part 模型 |
| 异步流式 | A2A 的 StreamingMessage |
| Run 状态机 | A2A 的 Task 生命周期 |
| `/.well-known/agent.json` | A2A 的 Agent Card |

### 迁移路径

```bash
# 1. 安装 A2A 适配器
pip install a2a-sdk[acp-adapter]

# 2. 运行迁移工具
a2a-migrate --from acp --source ./my_acp_agent --output ./my_a2a_agent
```

迁移工具会自动转换端点路径、Part 字段名、状态枚举。

### 与 MCP / AG-UI 的协同

```
┌──────────┐
│   UI     │
└────┬─────┘
     │ AG-UI（流式状态到前端）
┌────▼─────┐
│  Agent   │ ───── MCP ────> 工具
└────┬─────┘
     │ A2A (含 ACP 特性)
┌────▼─────┐
│  Agent   │
└──────────┘
```

## 代码示例

```python
# ACP 客户端调用示例（已被 A2A SDK 取代，仅供历史参考）
import httpx

async with httpx.AsyncClient(base_url="http://agent-b:8000") as client:
    # 1. 发现
    manifest = (await client.get("/.well-known/agent.json")).json()

    # 2. 启动运行
    run = (await client.post(
        "/agents/research-agent/runs",
        json={
            "parts": [
                {"type": "text", "text": "总结这篇文章"},
                {"type": "image", "url": "https://..."},
            ]
        },
    )).json()
    run_id = run["run_id"]

    # 3. SSE 流式接收
    async with client.stream(
        "GET", f"/agents/research-agent/runs/{run_id}/stream"
    ) as response:
        async for line in response.aiter_lines():
            print(line)
```

## 时间线

| 时间 | 事件 |
|------|------|
| 2025-03 | IBM Research 发布 ACP v0.1 |
| 2025-04 | A2A 发布，定位与 ACP 高度重合 |
| 2025-06 | ACP v0.2 加入 OpenTelemetry 追踪 |
| 2025-08 | IBM 与 Linux Foundation 决定合并，ACP 并入 A2A |
| 2025-09 | A2A v0.3 RC1 吸收 ACP 的多模态 Part 模型 |
| 2026-Q1 | A2A v1.0 完成 ACP 全部特性迁移 |

## 参考资料

- [BeeAI ACP GitHub](https://github.com/i-am-bee/agent-communication-protocol)（已存档）
- [BeeAI Platform](https://github.com/i-am-bee/beeai-platform)
- [OSSA 协议调研](https://openstandardagents.org/research/agent-communication-protocol-survey/)
- [[../A2A 协议/A2A 原理|A2A 原理]] — ACP 的继任者