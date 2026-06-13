---
title: AG-UI 协议（Agent-User Interaction Protocol）
aliases: [AG-UI, Agent-User Interaction]
tags:
  - protocol
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-13
spec_url: https://github.com/CopilotKit/CopilotKit
maintainer: CopilotKit
transport: HTTP/SSE / WebSocket
---

# AG-UI 协议（Agent-User Interaction Protocol）

> CopilotKit 于 2025 年发布。专注于 Agent 到前端的**实时流式**通信，让 UI 能跟随 Agent 思考过程动态更新。

## 协议概览

| 项目 | 详情 |
|------|------|
| 全称 | Agent-User Interaction Protocol (AG-UI) |
| 提出者 | CopilotKit |
| 规范链接 | [CopilotKit GitHub](https://github.com/CopilotKit/CopilotKit) |
| 传输层 | HTTP/SSE / WebSocket |
| 消息格式 | JSON 事件流 |
| 方向 | 前端（Agent→UI） |

## 设计动机

在 AG-UI 出现之前，前端集成 Agent 通常用以下方式之一：

1. **轮询**：每 1s 拉取 Agent 状态 → 延迟高、资源浪费
2. **WebSocket 自定义**：灵活但每个项目要重新设计事件协议
3. **Server-Sent Events**：单向流式但缺少事件类型规范

AG-UI 提供：

| 协议 | 方向 | 用途 |
|------|------|------|
| [[../MCP 协议/MCP 原理\|MCP]] | Agent→工具 | 调用外部能力 |
| [[../A2A 协议/A2A 原理\|A2A]] | Agent↔Agent | Agent 间协作 |
| **AG-UI** | **Agent→前端** | **实时状态流式传输到 UI** |

## 核心架构

```
┌─────────────────┐       AG-UI 事件流       ┌──────────────┐
│                 │ ──────────────────────> │              │
│  Agent Backend  │   (SSE / WebSocket)      │   Frontend   │
│                 │ <────────────────────── │   (React)    │
└─────────────────┘     用户操作 / 表单输入    └──────────────┘
```

- **单向**：Backend → Frontend 推送事件流
- **反向**：Frontend → Backend 发送用户输入、审批、取消指令
- **嵌入性**：AG-UI 没有独立的发现机制，作为应用层嵌入到具体产品中

## 关键概念

- 基于 HTTP/SSE 和 WebSocket（默认 SSE，需要双向时升级 WebSocket）
- JSON 事件格式，每个事件有 `type` + `data` 字段
- 实时流式 Agent 状态到前端（token 流、工具调用、状态切换）
- 嵌入式使用（无需独立发现机制）

## 消息类型（事件）

AG-UI 定义了 16 种标准事件，覆盖完整的 Agent 生命周期：

| 事件类型 | 方向 | 用途 |
|---------|------|------|
| `RUN_STARTED` | Backend→UI | Agent 开始运行 |
| `RUN_FINISHED` | Backend→UI | Agent 完成（含 token 用量） |
| `RUN_ERROR` | Backend→UI | Agent 报错 |
| `STEP_STARTED` | Backend→UI | 单步推理开始 |
| `STEP_FINISHED` | Backend→UI | 单步完成 |
| `TEXT_MESSAGE_START` | Backend→UI | 文本片段开始 |
| `TEXT_MESSAGE_CONTENT` | Backend→UI | 文本 token 流式 |
| `TEXT_MESSAGE_END` | Backend→UI | 文本片段结束 |
| `TOOL_CALL_START` | Backend→UI | 工具调用开始（含参数） |
| `TOOL_CALL_END` | Backend→UI | 工具调用结束（含结果） |
| `TOOL_CALL_CHUNK` | Backend→UI | 流式工具参数（长参数用） |
| `STATE_SNAPSHOT` | Backend→UI | 完整状态快照 |
| `STATE_DELTA` | Backend→UI | 状态增量（JSON Patch） |
| `MESSAGES_SNAPSHOT` | Backend→UI | 消息历史快照 |
| `CUSTOM` | Backend→UI | 应用自定义事件 |
| `RAW` | Backend→UI | 透传事件 |

## 交互流程

### 标准流式响应

```
Backend                              Frontend
   │                                    │
   │ ─── RUN_STARTED ─────────────────> │ 显示"Agent 思考中"
   │ ─── TEXT_MESSAGE_START ──────────> │
   │ ─── TEXT_MESSAGE_CONTENT ────────> │ 逐 token 显示回复
   │ ─── TEXT_MESSAGE_CONTENT ────────> │
   │ ─── TOOL_CALL_START ─────────────> │ 显示"正在调用工具"
   │ ─── TOOL_CALL_END ───────────────> │ 显示工具结果
   │ ─── TEXT_MESSAGE_END ────────────> │
   │ ─── RUN_FINISHED ────────────────> │ 隐藏加载状态
```

### Human-in-the-Loop 中断模式

```
Backend                              Frontend
   │                                    │
   │ ─── TOOL_CALL_START ─────────────> │
   │   { name: "delete_record",        │
   │     destructive: true }            │
   │                                    │
   │      [前端弹出确认对话框]           │
   │                                    │
   │ <── TOOL_CALL_APPROVE ──────────── │ 用户点确认
   │ ─── TOOL_CALL_END ───────────────> │ 继续执行
```

## 安全考量

- **鉴权**：所有 AG-UI 端点应验证用户身份（OAuth / Session）
- **事件过滤**：不要把内部 secrets 推送到前端
- **速率限制**：防止恶意前端触发事件风暴
- **跨域**：生产环境应限制允许的 origin（CORS）
- **审计**：所有用户操作事件应记录到日志

## 与其他协议的关系

```
            ┌──────────┐
            │   User   │
            └────┬─────┘
                 │ AG-UI
            ┌────▼─────┐         MCP         ┌──────────┐
            │ Frontend │ ──────────────────> │  Tools   │
            └────▲─────┘                     └──────────┘
                 │                              ▲
                 │ AG-UI                        │ MCP
            ┌────┴─────┐                       │
            │  Agent   │ ──── A2A ────> Agent ─┘
            │ Backend  │
            └──────────┘
```

| 协议 | 方向 | 解决层 |
|------|------|--------|
| MCP | 后端 → 工具 | 能力层 |
| A2A | Agent → Agent | 协作层 |
| **AG-UI** | **Agent → 前端** | **呈现层** |

三者层次分明：MCP 给 Agent 能力、A2A 让 Agent 协作、AG-UI 把 Agent 状态送给用户看。

## 适用场景

- 构建 Agent 驱动的 Web 应用（Copilot、Research Assistant）
- 实时展示 Agent 推理过程（"AI 思考中..." 动画）
- 用户交互式 Agent 工作流（Human-in-the-Loop 审批）
- IDE 类工具的侧边栏对话（Cursor、Continue）

## 代码示例

### Backend（Python FastAPI + SSE）

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio, json

app = FastAPI()

async def ag_ui_stream(prompt: str):
    # 事件 1：Run 开始
    yield f"data: {json.dumps({'type': 'RUN_STARTED', 'runId': 'r-001'})}\n\n"

    # 事件 2-4：流式文本
    for token in ["你好", "，", "我是", "助手"]:
        yield f"data: {json.dumps({'type': 'TEXT_MESSAGE_CONTENT', 'delta': token})}\n\n"
        await asyncio.sleep(0.05)

    # 事件 5：完成
    yield f"data: {json.dumps({'type': 'RUN_FINISHED', 'runId': 'r-001'})}\n\n"

@app.get("/agent/stream")
async def stream(prompt: str):
    return StreamingResponse(
        ag_ui_stream(prompt),
        media_type="text/event-stream",
    )
```

### Frontend（React + CopilotKit）

```tsx
import { CopilotKit } from "@copilotkit/react-core";
import { CopilotChat } from "@copilotkit/react-ui";

function App() {
  return (
    <CopilotKit runtimeUrl="/agent/stream">
      <CopilotChat
        labels={{
          title: "研究助手",
          initial: "你好，需要我帮你查什么？",
        }}
        // 自动渲染 RUN_STARTED → 加载动画
        // 自动渲染 TEXT_MESSAGE_CONTENT → 逐字流式
        // 自动渲染 TOOL_CALL → 工具卡片
      />
    </CopilotKit>
  );
}
```

## 参考资料

- [CopilotKit](https://github.com/CopilotKit/CopilotKit)
- [AG-UI 事件规范](https://docs.copilotkit.ai/ag-ui)
- 相关协议：[[../MCP 协议/MCP 原理|MCP 原理]]、[[../A2A 协议/A2A 原理|A2A 原理]]