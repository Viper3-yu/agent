---
title: Claude Code 命令参考
aliases: [Claude Code CLI, Claude Code Commands, Claude Code Reference]
tags:
  - framework
  - topic/claude-code
  - topic/cli
  - status/reviewed
created: 2026-06-10
updated: 2026-06-10
language: TypeScript
version: ""
repo: ""
license: Proprietary (Anthropic)
---

# Claude Code 命令参考

> Claude Code CLI 的完整命令参考。涵盖斜杠命令、键盘快捷键和 CLI 启动参数。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官方文档 | https://docs.anthropic.com/en/docs/claude-code |
| 所属产品 | Anthropic Claude Code |
| 安装方式 | `npm install -g @anthropic-ai/claude-code` |
| 支持平台 | macOS, Linux, Windows (WSL/Git Bash) |
| 前置要求 | Node.js 18+, Anthropic API key 或 Claude Pro/Max 订阅 |

## 核心架构

Claude Code 是一个在终端中运行的**智能编程代理**，通过自然语言与开发者交互，能够读写文件、执行命令、搜索代码、管理 Git 操作等。

```
┌──────────────────────────────────────────┐
│              Claude Code CLI              │
│                                          │
│  输入方式:                                │
│  ├── 交互式 REPL（默认）                  │
│  ├── 单次模式 (--print)                   │
│  └── 管道输入 (echo "..." | claude)       │
│                                          │
│  核心能力:                                │
│  ├── 文件读写 (Read, Write, Edit)        │
│  ├── 命令执行 (Bash)                      │
│  ├── 代码搜索 (Grep, Glob)               │
│  ├── Git 操作                             │
│  └── MCP 工具扩展                         │
└──────────────────────────────────────────┘
```

## 快速上手

### 安装

```bash
# 全局安装
npm install -g @anthropic-ai/claude-code

# 启动
claude

# 或直接使用 npx
npx @anthropic-ai/claude-code
```

### 最小示例

```bash
# 交互式启动
claude

# 单次问答
claude -p "解释这段代码的作用"

# 管道输入
cat main.py | claude -p "找出这段代码中的 bug"

# 继续上次对话
claude --resume
```

## 关键概念

### 斜杠命令（Slash Commands）

在交互式 REPL 中输入 `/` 可查看所有可用命令：

| 命令 | 功能 | 说明 |
|------|------|------|
| `/init` | 初始化项目 | 扫描代码库生成 `CLAUDE.md` 项目上下文文件 |
| `/compact` | 压缩上下文 | 总结当前对话以减少 Token 使用，保留关键信息 |
| `/clear` | 清空对话 | 完全清除当前对话历史，从头开始 |
| `/cost` | 查看费用 | 显示当前会话的 Token 使用量和费用估算 |
| `/doctor` | 诊断检查 | 检查 Claude Code 安装、配置和连接状态 |
| `/help` | 帮助信息 | 显示可用命令和使用说明 |
| `/permissions` | 权限管理 | 管理工具权限设置（允许/禁止文件写入、Bash 等） |
| `/review` | 代码审查 | 请求对最近更改进行代码审查 |
| `/terminal-setup` | 终端配置 | 配置终端集成（Shift+Enter 换行等） |
| `/config` | 配置查看 | 查看或修改配置设置 |
| `/model` | 切换模型 | 在可用模型之间切换（如 Sonnet 4, Opus 4） |
| `/vim` | Vim 模式 | 切换 Vim 键绑定模式 |
| `/login` | 登录 | 使用 Anthropic 账户/API key 认证 |
| `/logout` | 登出 | 退出当前会话认证 |
| `/bug` | 报告问题 | 向 Anthropic 报告 Claude Code 的问题 |
| `/mcp` | MCP 管理 | 查看和管理 MCP Server 连接状态 |

### CLI 启动参数

```bash
claude [options] [prompt]
```

| 参数 | 说明 | 示例 |
|------|------|------|
| `--print`, `-p` | 单次模式，输出结果后退出 | `claude -p "写一个 hello world"` |
| `--resume`, `-r` | 继续上一次对话 | `claude --resume` |
| `--continue`, `-c` | 继续最近的对话（不询问选择） | `claude --continue` |
| `--verbose` | 显示详细输出（包括工具调用过程） | `claude --verbose` |
| `--model` | 指定使用的模型 | `claude --model claude-sonnet-4-20250514` |
| `--output-format` | 输出格式（text, json, stream-json） | `claude -p "..." --output-format json` |
| `--max-turns` | 限制代理执行的最大轮数 | `claude -p "..." --max-turns 5` |
| `--allowedTools` | 指定允许使用的工具列表 | `claude --allowedTools "Read,Write,Grep"` |
| `--disallowedTools` | 指定禁止使用的工具列表 | `claude --disallowedTools "Bash"` |
| `--system-prompt` | 自定义系统提示（附加到默认提示后） | `claude --system-prompt "你是 Rust 专家"` |
| `--append-system-prompt` | 追加系统提示 | `claude --append-system-prompt "用中文回答"` |
| `--permission-mode` | 权限模式 | `claude --permission-mode plan` |
| `--mcp-config` | 指定 MCP 配置文件 | `claude --mcp-config .mcp.json` |
| `--debug` | 启用调试模式 | `claude --debug` |
| `--version`, `-v` | 显示版本号 | `claude --version` |
| `--help`, `-h` | 显示帮助信息 | `claude --help` |

### 常用 CLI 组合

```bash
# 单次代码审查
git diff | claude -p "审查这些更改，找出潜在问题"

# 生成提交消息
git diff --staged | claude -p "生成一个 conventional commit 格式的提交消息"

# 批量处理文件
claude -p "将所有 .js 文件转换为 TypeScript" --max-turns 20

# JSON 输出（适合管道处理）
claude -p "列出所有 TODO 注释" --output-format json

# 使用特定模型
claude --model claude-opus-4-20250514 -p "分析这个算法的时间复杂度"

# 禁止危险操作
claude --disallowedTools "Bash" -p "重构这个函数"
```

### 键盘快捷键

在交互式 REPL 中可用的快捷键：

| 快捷键 | 功能 |
|--------|------|
| `Enter` | 发送消息（单行） |
| `Shift+Enter` | 换行（需先运行 `/terminal-setup`） |
| `Option+T` / `Alt+T` | 切换扩展思考（Extended Thinking） |
| `Ctrl+C` | 中断当前操作 |
| `Ctrl+D` | 退出 Claude Code |
| `Ctrl+O` | 显示思考输出（Verbose thinking） |
| `↑` / `↓` | 浏览输入历史 |
| `Tab` | 自动补全文件路径 |
| `Esc` | 取消当前输入 |

### 权限模式

| 模式 | 说明 |
|------|------|
| `default` | 默认模式，高危操作需要确认 |
| `plan` | 规划模式，只读不写，适合代码分析 |
| `bypass` | 跳过所有权限确认（谨慎使用） |

### 配置文件

| 文件 | 路径 | 用途 |
|------|------|------|
| `CLAUDE.md` | 项目根目录 | 项目级指令和上下文 |
| `settings.json` | `.claude/settings.json` | 项目级设置（hooks、权限等） |
| `settings.json` | `~/.claude/settings.json` | 用户级全局设置 |
| `.mcp.json` | 项目根目录 | MCP Server 配置 |

## 典型使用场景

### 1. 代码审查

```bash
# 审查未提交的更改
git diff | claude -p "审查这些更改，关注安全问题和代码质量"

# 审查 PR
gh pr diff 123 | claude -p "审查这个 PR 的变更"
```

### 2. 代码重构

```bash
# 交互式重构
claude "将 src/utils.js 重构为 TypeScript，保持所有功能不变"

# 批量重构
claude -p "将所有 class 组件转换为函数式组件" --max-turns 30
```

### 3. 调试辅助

```bash
# 分析错误日志
cat error.log | claude -p "分析这个错误日志，找出根本原因"

# 测试失败分析
npm test 2>&1 | claude -p "分析测试失败的原因并提供修复建议"
```

### 4. 文档生成

```bash
# 生成 API 文档
claude -p "为 src/api/ 目录下的所有接口生成 OpenAPI 规范"

# 生成 README
claude "基于项目结构和代码生成一份 README.md"
```

### 5. Git 工作流

```bash
# 生成提交消息
git diff --staged | claude -p "用 conventional commits 格式生成提交消息"

# 分支管理
claude "创建一个 feature 分支，实现用户认证功能"
```

## 与同类对比

| 维度 | Claude Code | GitHub Copilot CLI | Cursor | Aider |
|------|-------------|-------------------|--------|-------|
| 交互方式 | 终端 REPL | 终端命令 | IDE GUI | 终端 REPL |
| 代码编辑 | 直接文件操作 | 命令生成 | GUI 编辑 | 直接文件操作 |
| Git 集成 | 内置 | 内置 | 内置 | 内置 |
| 多文件编辑 | 支持 | 有限 | 支持 | 支持 |
| MCP 支持 | 原生 | 无 | 无 | 无 |
| 权限控制 | 细粒度 | 无 | 无 | 有限 |
| 价格 | API 按量 / Pro 订阅 | $10-19/月 | $20/月 | API 按量 |

## 已知局限

- **上下文窗口限制**：长会话需要使用 `/compact` 压缩上下文
- **文件大小限制**：单次读取文件有行数限制（2000 行）
- **并行操作限制**：同一文件不能同时被多个操作修改
- **网络依赖**：需要稳定的网络连接调用 API
- **中文支持**：CLI 界面和文档主要为英文，中文交互体验有待提升
- **Windows 兼容**：需要 WSL 或 Git Bash，原生 Windows 支持有限

## 参考资料

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code CLI 参考](https://docs.anthropic.com/en/docs/claude-code/cli-usage)
- [Claude Code Hooks 文档](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [Claude Code 设置文档](https://docs.anthropic.com/en/docs/claude-code/settings)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)
