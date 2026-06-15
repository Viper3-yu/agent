---
title: Planning 项目库
aliases: [规划中项目, Planning Projects, Backlog]
tags:
  - MOC
  - project
  - status/reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Planning 项目库

> 已有想法但**尚未启动**的项目。处于调研、立项、待资源、待触发条件等状态。

## 状态约定

| 状态 | 含义 | 触发转入 Active |
|------|------|---------------|
| `backlog` | 仅一句话想法 | 立项 |
| `research` | 调研中（市场 / 技术 / 资源） | 立项 + 有最小方案 |
| `blocked` | 等待外部条件 | 条件解除 |
| `dropped` | 已放弃 | 永久（不转 Active） |

## 当前规划项目

<!-- 项目列表 -->

- （无）

## 立项流程

```
1. 在本 README 下加一行：- [ ] [[<项目名>/README|<项目名>]] — <一句话目标>
2. 创建文件夹：05-Projects/Planning/<项目名>/
3. 复制 Templates/tpl-项目 README.md 到该文件夹
4. 填写"项目目标 / 背景与动机 / 触发条件"
5. 触发条件达成 → 移动到 Active/
```

## 立项模板（最小）

```markdown
## 项目目标
<!-- 一句话 -->

## 触发条件
<!-- 什么情况下会启动？ -->

## 预研
<!-- 已经知道什么 / 还需调研什么 -->
```

## 相关资源

- → [[../Active/README|Active 项目]]
- → [[../Archive/README|已归档项目]]
- → [[../../00-MOC/项目看板|项目看板 MOC]]
- → [[../../00-MOC/KB 构建规范|KB 构建规范]] §三 笔记类型
