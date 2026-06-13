---
title: LangChain 快速上手
aliases:
  - LangChain
tags:
  - framework
  - status/reviewed
created: 2026-06-12
updated: 2026-06-12
language: Python, TypeScript
version: "2026"
repo: https://github.com/langchain-ai/langchain
license: MIT
---

# LangChain 快速上手

> 最流行的 LLM 应用开发框架

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://www.langchain.com |
| GitHub | https://github.com/langchain-ai/langchain |
| 语言 | Python, TypeScript |
| 最新版本 | 2026 |
| 许可证 | MIT |

## 核心架构

<!-- 待填写 -->

## 快速上手

### 安装

```bash
pip install langchain
```

### 最小示例

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model="gpt-4o")
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "{input}")
])
chain = prompt | llm
result = chain.invoke({"input": "Hello!"})
print(result.content)
```

## 关键概念

<!-- 待填写 -->

## 典型使用场景

<!-- 待填写 -->

## 与同类对比

<!-- 待填写 -->

## 已知局限

<!-- 待填写 -->

## 参考资料

- [LangChain 文档](https://python.langchain.com)
- [[../LangGraph/LangGraph|LangGraph]]（LangChain 生态内的 Agent 编排框架）
