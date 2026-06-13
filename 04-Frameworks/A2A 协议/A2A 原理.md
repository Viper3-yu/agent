---
title: A2A 协议（Agent-to-Agent Protocol）
aliases: [A2A, Agent2Agent, Agent-to-Agent]
tags:
  - framework
  - topic/agent
  - status/mature
created: 2026-06-12
updated: 2026-06-12
spec_url: https://github.com/google/A2A
maintainer: Google / Linux Foundation
transport: JSON-RPC 2.0 / gRPC / HTTP+JSON
---

# A2A 协议（Agent-to-Agent Protocol）

> Google 于 2025 年 4 月发布，现由 Linux Foundation 治理。A2A 是 Agent 间横向通信的开放标准，解决 Agent 发现、任务委派和协作问题。2025 年 8 月，IBM 的 [[../MCP 协议/ACP 原理|ACP]] 正式并入 A2A。

## 协议概览

| 项目 | 详情 |
|------|------|
| 全称 | Agent-to-Agent Protocol (A2A) |
| 提出者 | Google（现由 Linux Foundation 治理） |
| 规范链接 | [A2A 官方规范](https://github.com/google/A2A) |
| 传输层 | JSON-RPC 2.0 / gRPC / HTTP+JSON |
| 消息格式 | JSON-RPC 2.0 |
| 方向 | 水平（Agent↔Agent） |

## 设计动机

| 层级 | 协议 | 方向 |
|------|------|------|
| 工具访问 | [[../MCP 协议/MCP 原理|MCP]] | 垂直（模型→工具） |
| **Agent 间通信** | **A2A** | **水平（Agent↔Agent）** |
| 前端交互 | [[../AG-UI 协议/AG-UI 原理|AG-UI]] | 垂直（Agent→UI） |

## 核心架构

<!-- 待填写 -->

## 关键概念

### Agent Card
每个 A2A Server 在 `/.well-known/agent.json` 发布能力声明：
- 名称、URL、版本
- **Skills**：Agent 能做什么 → 详见 [[../Agent Skills/A2A Agent Skills|A2A Agent Skills]]
- 输入/输出 Schema
- 认证要求

### 任务生命周期

```
submitted → working → input-required → completed / failed / canceled
```

### 协议绑定
- JSON-RPC 2.0（主要）
- gRPC
- HTTP+JSON/REST

## 消息类型

| 消息类型 | 方向 | 用途 |
|---------|------|------|
| `tasks/send` | Client→Server | 发送任务消息 |
| `tasks/sendSubscribe` | Client→Server | 流式任务更新 |
| `tasks/get` / `tasks/list` | Client→Server | 查询任务状态 |
| `tasks/cancel` | Client→Server | 取消任务 |

## 交互流程

1. Client 发现 Agent Card（`/.well-known/agent.json`）
2. Client 解析 Skills，选择目标 Agent
3. Client 通过 `tasks/send` 发送任务
4. Server 返回 Task，状态为 `submitted`
5. Server 处理任务，状态变为 `working`
6. 若需要更多信息，状态变为 `input-required`
7. 任务完成，状态变为 `completed`（附结果）或 `failed`

## 安全考量

<!-- 待填写 -->

## 与其他协议的关系

- **MCP = 手**：Agent 调用工具（垂直）
- **A2A = 声**：Agent 间对话（水平）
- 生产系统两者兼用：MCP 连工具，A2A 连 Agent

## 代码示例

<!-- 待填写 -->

## 版本演进

| 版本 | 日期 | 亮点 |
|------|------|------|
| v0.1 | 2025-04 | Google 首发，50+ 合作伙伴 |
| v0.3 RC1 | 2026-02 | gRPC 支持，安全卡片签名 |
| v1.0 | 2026 | 正式版，纳入 ACP 特性 |

## 采纳情况

- Google ADK：原生支持
- CrewAI：2026 年添加 A2A 任务委派
- 150+ 组织贡献规范

## 参考资料

- [A2A 官方规范](https://github.com/google/A2A)
- [arXiv 论文](https://arxiv.org/html/2505.02279v1)
- [Tyk 详解](https://tyk.io/learning-center/agent-protocols-a-complete-guide-to-mcp-a2a-and-acp/)
