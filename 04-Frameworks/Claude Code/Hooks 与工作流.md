---
title: Hooks 与工作流
aliases: [Claude Code Hooks, Hooks, Claude Code Workflows]
tags:
  - framework
  - topic/claude-code
  - status/seed
created: 2026-06-12
updated: 2026-06-12
language: TypeScript/JavaScript
version: ""
repo: ""
license: Proprietary (Anthropic)
---

# Hooks 与工作流

> Claude Code 的 Hooks 系统和自动化工作流。Hooks 允许在 Claude Code 的工具调用生命周期中插入自定义脚本，实现自动化检查、格式化、安全验证等。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官方文档 | https://docs.anthropic.com/en/docs/claude-code/hooks |
| 所属产品 | Claude Code (Anthropic) |
| 配置方式 | `.claude/settings.json` |
| 支持平台 | macOS, Linux, Windows (WSL) |
| 触发时机 | 工具调用前/后、会话结束时 |

## 核心架构

Claude Code Hooks 是基于**事件驱动**的脚本执行系统，在 Claude Code 的工具调用生命周期中提供三个关键挂载点：

```
工具调用流程:
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  PreToolUse  │ ──> │  Tool 执行   │ ──> │ PostToolUse  │
│  (工具调用前) │     │             │     │  (工具调用后) │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
                                               v
                                        ┌─────────────┐
                                        │    Stop      │
                                        │ (会话结束时)  │
                                        └─────────────┘
```

| Hook 类型 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| **PreToolUse** | 工具执行前 | 参数验证、权限检查、输入清洗 |
| **PostToolUse** | 工具执行后 | 自动格式化、结果验证、日志记录 |
| **Stop** | 会话结束时 | 最终检查、清理、报告生成 |

## 快速上手

### 配置文件位置

Hooks 在 `.claude/settings.json` 中配置，支持三个级别：

| 级别 | 路径 | 作用域 |
|------|------|--------|
| 项目级 | `<project>/.claude/settings.json` | 当前项目 |
| 用户级 | `~/.claude/settings.json` | 所有项目 |
| 企业级 | 由管理员管理 | 整个组织 |

### 最小示例

在 `.claude/settings.json` 中添加一个 Hook，在文件写入后自动格式化：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

### 完整配置示例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/check_dangerous_commands.py"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      },
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix $CLAUDE_FILE_PATH"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/final_review.py"
          }
        ]
      }
    ]
  }
}
```

## 关键概念

### Matcher（匹配器）

Matcher 决定 Hook 对哪些工具生效：

```json
// 匹配所有工具
"matcher": "*"

// 匹配特定工具（正则表达式）
"matcher": "Write|Edit"

// 匹配单个工具
"matcher": "Bash"
```

### 环境变量

Hook 脚本可以访问以下环境变量：

| 变量名 | 说明 | 可用时机 |
|--------|------|---------|
| `CLAUDE_FILE_PATH` | 正在操作的文件路径 | PostToolUse |
| `CLAUDE_TOOL_NAME` | 当前工具名称 | PreToolUse, PostToolUse |
| `CLAUDE_TOOL_INPUT` | 工具输入 JSON | PreToolUse |
| `CLAUDE_TOOL_OUTPUT` | 工具输出 JSON | PostToolUse |

### Hook 脚本规范

Hook 脚本通过**退出码**和 **stdout** 与 Claude Code 交互：

| 退出码 | 含义 | 行为 |
|--------|------|------|
| `0` | 成功 | 继续执行 |
| `2` | 阻止 | 阻止工具调用（PreToolUse）或标记为失败（PostToolUse） |
| 其他 | 错误 | 记录错误，继续执行 |

### PreToolUse 阻止机制

PreToolUse Hook 可以通过退出码 `2` 阻止危险操作：

```python
#!/usr/bin/env python3
"""阻止危险的 Bash 命令"""
import sys
import os
import json

tool_input = json.loads(os.environ.get("CLAUDE_TOOL_INPUT", "{}"))
command = tool_input.get("command", "")

dangerous_patterns = ["rm -rf /", "mkfs", "dd if=", "> /dev/sda"]

for pattern in dangerous_patterns:
    if pattern in command:
        print(f"BLOCKED: 危险命令 '{pattern}' 被拦截", file=sys.stderr)
        sys.exit(2)

sys.exit(0)
```

## 典型使用场景

### 1. 代码自动格式化

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

### 2. 安全命令检查

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/safety_check.py"
          }
        ]
      }
    ]
  }
}
```

### 3. 自动测试触发

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/run_related_tests.py"
          }
        ]
      }
    ]
  }
}
```

### 4. Git 提交前检查

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/pre_commit_check.py"
          }
        ]
      }
    ]
  }
}
```

### 5. 日志和审计

```python
#!/usr/bin/env python3
"""记录所有工具调用到审计日志"""
import os
import json
from datetime import datetime

log_entry = {
    "timestamp": datetime.now().isoformat(),
    "tool": os.environ.get("CLAUDE_TOOL_NAME"),
    "input": os.environ.get("CLAUDE_TOOL_INPUT"),
}

with open(".claude/audit.log", "a") as f:
    f.write(json.dumps(log_entry) + "\n")
```

## 与同类对比

| 维度 | Claude Code Hooks | Git Hooks | CI/CD Hooks | Pre-commit |
|------|------------------|-----------|-------------|------------|
| 触发点 | AI 工具调用生命周期 | Git 操作 | 代码推送 | Git commit |
| 粒度 | 单个工具调用 | 整个 Git 操作 | 整个流水线 | 整个 commit |
| 灵活度 | 高（可读取上下文） | 中 | 高 | 中 |
| 阻止能力 | 可阻止单个操作 | 可阻止整个操作 | 可阻止合并 | 可阻止提交 |
| 典型用途 | AI 行为约束 | 代码质量 | 部署安全 | 提交质量 |

## 已知局限

- **性能开销**：每个工具调用都会触发 Hook 脚本，频繁调用可能影响响应速度
- **调试困难**：Hook 脚本的错误不容易在 Claude Code 界面中看到
- **跨平台兼容**：Shell 命令在不同操作系统上可能有差异
- **依赖管理**：Hook 脚本依赖的工具需要预先安装
- **顺序依赖**：多个 Hook 的执行顺序可能影响结果

## 参考资料

- [Claude Code Hooks 官方文档](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [Claude Code CLI 参考](https://docs.anthropic.com/en/docs/claude-code/cli-usage)
- [Claude Code 设置文档](https://docs.anthropic.com/en/docs/claude-code/settings)
