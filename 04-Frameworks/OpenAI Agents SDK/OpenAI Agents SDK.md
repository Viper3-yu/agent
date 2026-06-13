---
title: OpenAI Agents SDK
aliases: [OpenAI Agents SDK, Responses API, OpenAI Agent Framework]
tags:
  - framework
  - topic/agent
  - status/seed
created: 2026-06-12
updated: 2026-06-12
language: Python, TypeScript
version: "2026"
repo: https://github.com/openai/openai-agents-python
license: MIT
---

# OpenAI Agents SDK

> OpenAI 的官方 Agent 框架，强调最小原语（Agent、Handoff、Guardrail），快速组装多 Agent 系统。与 Responses API 深度绑定。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://platform.openai.com/docs/guides/agents-sdk |
| GitHub | https://github.com/openai/openai-agents-python |
| 语言 | Python, TypeScript |
| 最新版本 | 2026 |
| 许可证 | MIT |

## 核心架构

三大最小原语：
- **Agent**：封装模型 + 指令 + 工具
- **Handoff**：Agent 间任务交接
- **Guardrail**：三级安全护栏（输入/输出/全局）
- Sandbox 执行：隔离代码运行环境
- 语音支持：实时语音 Agent

## 快速上手

### 安装

```bash
pip install openai-agents
```

### 最小示例

```python
from agents import Agent, Runner

agent = Agent(
    name="Assistant",
    instructions="You are a helpful assistant.",
    model="gpt-4o"
)

result = Runner.run_sync(agent, "Hello!")
print(result.final_output)
```

## 关键概念

### Agent

封装模型、指令和工具的最小执行单元。

### Handoff

Agent 间任务交接机制，支持线性 Handoff 链。

### Guardrail

三级安全护栏：输入 Guardrail、输出 Guardrail、全局 Guardrail，保障 Agent 行为安全。

## 典型使用场景

1. **轻量级 Agent 交接链**: 多个 Agent 按序处理任务
2. **GPT-5-Codex 工作流**: 代码生成和审查流程
3. **GitHub / VS Code 生态集成**: 与开发工具深度集成

## 与同类对比

| 特性 | OpenAI Agents SDK | Claude Agent SDK | Google ADK |
|------|-------------------|------------------|------------|
| 模型绑定 | 仅 OpenAI | 仅 Claude | Gemini 优化 |
| 安全护栏 | 三级 Guardrail | Hook 生命周期 | 图引擎控制 |
| 语音支持 | 实时语音 Agent | 不支持 | 双向音视频 |
| 多语言 | Python, TS | Python, TS | Python, TS, Java, Go, Kotlin |

## 已知局限

- 仅支持 OpenAI 模型
- 暂不支持 A2A 跨厂商互操作
- 与 Responses API 深度绑定，迁移成本高

## 参考资料

- [OpenAI Agents SDK 文档](https://platform.openai.com/docs/guides/agents-sdk)
- [[../Claude Agent SDK/Claude Agent SDK|与 Claude Agent SDK 对比]]
