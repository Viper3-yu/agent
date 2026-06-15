---
title: 不同模型 Prompt 差异
aliases: [Model-Specific Prompting, GPT vs Claude vs Gemini, 跨模型 Prompt]
tags:
  - prompt
  - topic/prompt
  - status/draft
created: 2026-06-15
updated: 2026-06-15
---

# 不同模型 Prompt 差异

> 把同一个任务丢给 GPT / Claude / Gemini / 国产模型，**Prompt 写法需要针对性调整**。这篇是实战经验沉淀，帮"已经会用 CoT 的人"少踩 80% 的跨模型坑。

## 为什么这个文件存在

- 03-Prompt/技术/ 下的 [[../../Chain of Thought|Chain of Thought]] / [[../../Few-Shot Learning|Few-Shot]] / [[../../Self-Consistency|Self-Consistency]] / [[../../Tree of Thought|Tree of Thought]] / [[../../ReAct Prompting|ReAct]] / [[../../System Prompt 设计|System Prompt 设计]] 都是**模型无关**的——教的是"通用 prompt 技巧"
- 但实战发现：**同一段 prompt 在不同模型上的成功率可以差 30%+**，因为各家训练数据 / RLHF 偏好 / system prompt 设计哲学不一样
- 本文件收"针对具体模型的 prompt 适配"

## 核心思想

不同模型的"偏好"差异可归为 4 个维度：

| 维度 | 解释 | 影响 |
|------|------|------|
| **指令遵循精度** | 多复杂指令能 100% 执行 | 决定 system prompt 长度上限 |
| **格式稳定性** | JSON / XML / Markdown 输出的一致性 | 决定要不要加 few-shot |
| **CoT 自发度** | 不显式说也自己思考的概率 | 决定要不要加 "think step by step" |
| **工具调用能力** | 原生 tool use 的成熟度 | 决定是写 function calling 还是 ReAct 文本 |

## 实战差异速查

### 1. 系统提示词风格

| 模型 | 推荐风格 | 避免 |
|------|---------|------|
| **GPT-4o / GPT-5** | 简洁、结构化、bullet 多 | 长段落（容易"忽略"） |
| **Claude Sonnet / Opus** | 详尽、有 reasoning、明确边界 | 太短（容易"自由发挥"） |
| **Gemini 1.5/2.0 Pro** | 例子驱动、few-shot 多 | 纯文字指令 |
| **Qwen 2.5/3** | 简洁 + 中文优先 | 英文 prompt 翻译感重 |
| **DeepSeek-V3/R1** | 强 CoT、可加 "let's reason" | 不适合小模型改写 |
| **Kimi K2** | 长上下文友好、文件型 prompt | 简短精炼 |

### 2. CoT 触发方式

| 模型 | 默认行为 | 显式触发词 |
|------|---------|----------|
| **GPT-4o / o1** | o1 默认深度 CoT；4o 不显式 | "Let's think step by step" |
| **Claude** | Sonnet 4+ 默认中等 CoT | "Think carefully before answering" |
| **Gemini Pro** | 默认轻度 CoT | "Show your reasoning" |
| **DeepSeek-R1** | **默认就强 CoT** | 不需要，反而要 `<think>` 标签控制 |
| **o1 / Claude thinking** | **长 CoT 是特性** | 加 trigger 反而打扰 |

### 3. 工具调用

| 模型 | 原生 tool use | 备选写法 | 备注 |
|------|:---:|---------|------|
| **GPT-4o+** | ✅ | ReAct 文本 | 严格 schema |
| **Claude 3.5+** | ✅ | ReAct 文本 | 工具描述喜欢"功能 + 例子" |
| **Gemini 1.5+** | ✅ | ReAct 文本 | function calling 略弱于 OpenAI |
| **Qwen 2.5+** | ✅ | ReAct 文本 | 国产里最稳 |
| **DeepSeek-V3** | ⚠️ 部分 | ReAct 文本 | 文本调用更稳 |
| **Llama 3.3 / 3.1** | ❌ | ReAct 文本 | 必须自己解析 |

**关键经验**：**多模型 fallback 时用 ReAct 文本最稳**（不依赖任何一家的 tool use 协议），MCP 协议出来后这个问题缓解了。

### 4. 长上下文策略

| 模型 | 实际有效长度 | Lost-in-the-Middle 严重度 | 实战建议 |
|------|:---:|:---:|------|
| **Claude 3.5/4 Sonnet** | ~200K | 轻 | 关键信息放首尾 |
| **GPT-4o** | ~128K | 中 | 关键信息放首 |
| **Gemini 1.5 Pro** | ~1M（理论） | 中 | 实测 ~500K 后退化 |
| **Gemini 3.1 Pro** | ~2M | 轻 | 谷歌专门优化过 |
| **Qwen 2.5-1M** | ~1M | 中 | YaRN 扩展 |
| **Kimi K2** | ~200K | 轻 | 长文友好 |

参考：[[../../../01-Concepts/基础理论/Token 与上下文窗口]] / [[../../../01-Concepts/基础理论/Token 与上下文窗口#Lost-in-the-Middle|Lost-in-the-Middle 现象]]

### 5. 安全护栏（实战差异）

| 模型 | 拒答严格度 | 越狱抵抗 | 实战 |
|------|:---:|:---:|------|
| **Claude** | 高 | 强 | 偶尔"过度拒绝" |
| **GPT-4o** | 中 | 强 | 平衡 |
| **Gemini** | 中 | 中 | 偶尔漏 |
| **Llama 3.1** | 低 | 弱 | **自部署首选**（可定制） |

## 实战模板（跨模型通用化）

**GPT 4o 友好**：
```
You are a helpful assistant.

Task: <具体任务>
Output format: JSON with fields {x, y, z}
Constraints: 
- 不超过 200 字
- 必须包含 1 个例子
```

**Claude 友好**：
```
You are a <角色>. <一段角色描述>

Your goal is to <目标>.

Approach:
1. First, ...
2. Then, ...
3. Finally, ...

Constraints:
- ...

Output format:
<示例>

If you're unsure, say so explicitly. Don't guess.
```

**Gemini 友好**：
```
<任务描述>

Examples:
Input: ...
Output: ...

Now do:
Input: ...
```

## 反模式（不要这样写）

- ❌ "像人类一样思考" — 各家定义不一样
- ❌ "如果不知道就说不知道" — Claude / GPT 行为不同
- ❌ 写超长 system prompt（>2K token） — GPT 4o 容易"选择性忽略"
- ❌ "你是 ChatGPT" — 跨模型迁移会出戏
- ❌ 假设 tool use schema 通用 — OpenAI / Anthropic / Google 各自不同

## 评估方法（怎么知道 prompt 改对了）

1. **A/B 对比**：同一组 50 个测试用例，跑旧 / 新 prompt，看胜率
2. **Trace 看**："模型在哪个点开始走偏"——比"答得对不对"更有信号
3. **多模型交叉验证**：用 A 模型的输出喂给 B 模型评，反向发现 prompt 哪里没讲清

参考：[[../../评估与优化/Prompt 迭代方法论]] / [[../../../01-Concepts/基础理论/对齐与安全#评估方法]] 

## 相关笔记

- [[../../技术/Chain of Thought]] / [[../../技术/ReAct Prompting]] / [[../../技术/Self-Consistency]] — 通用技巧
- [[../../技术/System Prompt 设计]] — 写法
- [[../../../01-Concepts/基础理论/Token 与上下文窗口]] — 长上下文策略
- [[../../../01-Concepts/基础理论/对齐与安全]] — 安全护栏
- [[../../../02-Models/Anthropic/Claude Opus 4.8]] / [[../../../02-Models/Anthropic/Claude Sonnet 4.6]] / [[../../../02-Models/OpenAI/GPT-5]] / [[../../../02-Models/Google/Gemini 3.1 Pro]] — 具体模型笔记

## 参考资料

- OpenAI Cookbook "Prompt engineering" 章节
- Anthropic Prompt Engineering 文档
- Google "Prompting Guide 101"（白皮书）
- [system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools) — 业界实际 system prompt 仓库
