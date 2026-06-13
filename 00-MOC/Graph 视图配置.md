---
title: Graph 视图配置
aliases: [Graph View Setup, 知识图谱配置]
tags:
  - MOC
  - meta/setup
  - status/reviewed
created: 2026-06-13
updated: 2026-06-13
---

# Graph 视图配置指南

> 老大在 Obsidian 里手动操作。**不能纯靠 Claudian 改配置文件**——graph.json 在 Obsidian 启动时由 UI 写入。

## 1. 打开 Graph View

左侧栏 graph 图标（圆形节点）→ 出现全 vault 图谱。

**默认状态**：所有节点同色，孤立节点显示但灰一点。

## 2. 配置 Color Groups（颜色组）✅ 必做

### 操作步骤

```
1. 在 Graph View 内点右上角 ⚙️ 齿轮
2. 找到 "Color groups" 区块
3. 点 "+" 加 4 组
4. 逐组填：
```

### 4 组推荐配色

| Query | RGB | 色名 | 含义 |
|-------|-----|------|------|
| `path:00-MOC` | `#d29922` | 🟡 金 | MOC 入口 |
| `path:01-Concepts` | `#a371f7` | 🟣 紫 | 概念笔记 |
| `path:02-Models` | `#58a6ff` | 🔵 蓝 | 模型笔记 |
| `path:03-Prompt` | `#3fb950` | 🟢 绿 | Prompt 笔记 |
| `path:04-Frameworks` | `#f85149` | 🔴 红 | 框架/协议 |
| `path:05-Projects` | `#ff7b72` | 🟠 橙 | 项目 |
| `path:06-Resources` | `#8b949e` | ⚫ 灰 | 资源 |
| `path:Clippings` | `#6e7681` | ⚪ 浅灰 | 原始资料 |
| `#status/seed` | 半透明白 | 白 | 占位未填 |
| `#status/archived` | `#5a5a5a` | 深灰 | 已归档 |

> 颜色用 hex 码，Obsidian 自带拾色器。

### 进阶：按 topic 标签

如果想更细的颜色（如区分 Agent vs LLM theory vs Planning），用 tag 查询：

```
"#topic/llm"     → 紫色
"#topic/agent"   → 蓝色
"#topic/planning"→ 青色
"#topic/safety"  → 红色
```

> Obsidian 颜色组 query 语法：tag 写 `#xxx`、路径写 `path:xxx`、文件名写 `file:xxx`。

## 3. 调节点显示

同一 ⚙️ 面板下：

| 选项 | 推荐 | 原因 |
|------|------|------|
| **Show tags** | ✅ | 节点显示第一行 tag 信息 |
| **Show attachments** | ❌ | 太多附件节点会乱 |
| **Hide unresolved** | ❌ | 暂时保留，看断链 |
| **Show orphans** | ✅ | 看哪些没连上 |
| **Text fade multiplier** | `0` | 文字不淡出（默认）|
| **Node size multiplier** | `1.0 ~ 1.5` | 节点大小 |
| **Link force** | 默认 | 拉远/拉近 |

## 4. 安装 Juggl 插件 ⭐ 可选但强烈推荐

Juggl 是社区插件，给 Graph View 加**多维度着色**（按 degree、按 community、按 hub）。

### 安装

```
1. Obsidian → Settings → Community plugins → Browse
2. 搜 "Juggl"  → Install → Enable
```

### Juggl 着色方式

启用后，节点右键 → "Style" → 选维度：

| 维度 | 效果 |
|------|------|
| **Degree** | 节点颜色 = 引用数（hub 节点最红） |
| **Closeness** | 颜色 = 距离中心远近 |
| **Community** | 自动检测社群，社群内同色 |
| **Tag** | 优先用 Juggl 的 tag 配色 |
| **Custom** | 写自己的 color mapping |

> **Juggl 与 Obsidian 自带 Color Groups 不冲突**——可同时启用，Juggl 优先。

## 5. 验证配置

完成上述后，截图给我看效果，**老大觉得太花/太素可以再调**。

## 6. 备份 / 同步

`.obsidian/graph.json` 是 per-device 状态（已 gitignore）。**多设备使用时要在每台机器上独立配**。

如果想跨设备同步配置，可以**只跟踪 graph.json**——但会失去 per-device 灵活性。不建议。

## 7. 推荐 Obsidian 主题配合

- **Minimal**（你已装）—— 配合深色背景，节点色对比强
- **AnuPpuccin** —— 亮色 / 暗色都有柔和节点色
- **默认主题** —— 也行，但节点色被弱化
