# Obsidian Agent 知识库

> 老大的 LLM / Agent 主题**第二大脑**。按 Karpathy ["LLM Wiki"](Clippings/llm-wiki.md) 三层架构构建：原始资料 → 知识库 → 模式规范。

![vault stats](https://img.shields.io/badge/笔记-197-blue) ![status](https://img.shields.io/badge/status-active-green) ![license](https://img.shields.io/badge/license-private-red)

---

## 这是什么

一个**面向 LLM 与 Agent 工程**的 Obsidian vault，约 200 篇 markdown 笔记。结构按"原始资料 → wiki → schema"三层：

```
Clippings/      原始资料（剪藏文章、论文）—— 不可变
00-MOC/         地图与规范（MOC + KB 构建规范）
01-Concepts/    概念笔记（分词、嵌入、MoE、采样、量化、RAG、Agent 模式 …）
02-Models/      模型笔记（GPT、Claude、Gemini、DeepSeek、Qwen、Kimi、Grok、Llama、Mai …）
03-Prompt/      Prompt 工程笔记
04-Frameworks/  框架、协议、Agent 产品、评测工具
05-Projects/    个人项目
06-Resources/   微调、论文、博客等资源
Inbox/          临时收件箱（诊断报告、Claudian 草稿）
Templates/      Templater 模板
log.md          知识库变更日志
```

## 怎么用

- **读**：从 `00-MOC/LLM 模型地图.md` 或 `00-MOC/Agent 架构地图.md` 入手找主题
- **写**：用 `Templates/` 下的模板新建笔记，遵守 `00-MOC/KB 构建规范.md`
- **追溯**：所有重大变更记录在 `log.md`（按 `## [YYYY-MM-DD] <操作> | <标题>` 格式）

## 维护

- **作者**：老大 + Claudian（Obsidian 内的 AI 助手）
- **协作流程**：见 `00-MOC/KB 构建规范.md` §2（三大操作：Ingest / Query / Lint）
- **版本管理**：GitHub 私有仓库 [`Viper3-yu/agent`](https://github.com/Viper3-yu/agent)
- **Commit 约定**：`<类型>(<范围>): <描述>`（feat / fix / meta / chore / docs / refactor / lint）

## 三层架构速览

| 层 | 职责 | 位置 | 写者 |
|---|------|------|------|
| Raw Sources | 不可变原始资料 | `Clippings/` | 老大 |
| Wiki | LLM 生成的 markdown 知识库 | `01-` ~ `06-` 子目录 | Claudian（老大审） |
| Schema | 工作流与规范 | `00-MOC/KB 构建规范.md` + `log.md` | 共建 |

## 参考

- [[../Clippings/llm-wiki|Karpathy: LLM Wiki 模式]] —— 本知识库的理论基础
- [[../00-MOC/KB 构建规范|KB 构建规范]] —— 完整工作流、frontmatter、链接、协作边界
- [[../log|log.md]] —— 时间线，看最近 10 条用 `grep "^## \[" log.md | tail -10`
