---
title: Microsoft Agent Framework 1.0
aliases: [MS Agent Framework, Semantic Kernel, Microsoft Agent Framework]
tags:
  - framework
  - topic/agent
  - status/reviewed
created: 2026-06-12
updated: 2026-06-12
language: Python, .NET
version: "1.0"
repo: https://github.com/microsoft/semantic-kernel
license: MIT
---

# Microsoft Agent Framework 1.0

> 微软的 Agent 开发框架，整合 Semantic Kernel，支持群聊 + 图工作流，原生 MCP 和 A2A 支持，深度集成 Azure 生态。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://learn.microsoft.com/semantic-kernel |
| GitHub | https://github.com/microsoft/semantic-kernel |
| 语言 | Python, .NET |
| 最新版本 | 1.0 |
| 许可证 | MIT |

## 核心架构

整合 Semantic Kernel 的能力，提供群聊模式和图工作流两种 Agent 编排方式，原生支持 MCP 和 A2A 协议。

## 快速上手

### 安装

```bash
# Python
pip install semantic-kernel

# .NET
dotnet add package Microsoft.SemanticKernel
```

### 最小示例

<!-- 待填写 -->

## 关键概念

### 核心特性

| 特性 | 说明 |
|------|------|
| 语言 | Python, .NET |
| 多 Agent | 群聊 + 图工作流 |
| MCP 支持 | 原生 |
| A2A 支持 | 原生 |
| 人机协作 | 内置 Human-in-the-loop |
| 部署 | Azure / .NET 生态 |

### 群聊模式

多个 Agent 在同一会话中协作，支持 Human-in-the-loop 介入。

### 图工作流

用有向图定义 Agent 间的执行顺序和条件分支。

## 典型使用场景

1. **Azure / .NET 技术栈**: 深度集成 Azure 服务
2. **Human-in-the-loop**: 需要人工审批的企业场景
3. **Microsoft 365 集成**: Copilot 生态集成

## 与同类对比

| 特性 | MS Agent Framework | Google ADK | Claude Agent SDK |
|------|-------------------|-----------|-----------------|
| A2A 支持 | 原生 | 原生 | 暂不支持 |
| MCP 支持 | 原生 | 适配器 | 原生 |
| 人机协作 | 内置 | 无 | Hook |
| 部署 | Azure | GCP | 自托管/API |

## 已知局限

- 对非 Azure 用户吸引力有限
- Python SDK 功能不如 .NET SDK 完整
- 文档和社区生态仍在建设中

## 参考资料

- [Semantic Kernel](https://github.com/microsoft/semantic-kernel)
- [Zylos 多 Agent 协议研究](https://zylos.ai/research/2026-01-12-multi-agent-communication)
