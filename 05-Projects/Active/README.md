---
title: Active 项目看板
aliases: [进行中项目, Active Projects]
tags:
  - MOC
  - project
  - status/reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Active 项目看板

> 正在推进中的个人项目。每个项目一个文件夹，含 README + 子笔记。

## 状态约定

- 文件夹命名：`YYYY-MM-<项目代号>-<简短描述>` 或 `<项目代号>-<简短描述>`
- 例：`2026-06-claudian-vault-obsidian-pkm/`
- 每个项目根必须有 `README.md`（使用 `Templates/tpl-项目 README.md`）
- 状态：进行中 / 暂停 / 已完成（转 Archive）

## 当前项目

<!-- 项目列表：用 checklist 维护，每行一个项目链接 + 当前状态 -->

- （无）

## 添加新项目

```bash
# 1. 复制模板
cp Templates/tpl-项目\ README.md "05-Projects/Active/<项目名>/README.md"

# 2. 在本 README 的"当前项目"下加一行
- [ ] [[<项目名>/README|<项目名>]] — <一句话目标> (启动于 YYYY-MM-DD)
```

## 项目结构参考

```
05-Projects/Active/<项目名>/
├── README.md              # 必填：项目目标/背景/技术方案/里程碑
├── log.md                  # 进展日志
├── notes/                  # 自由笔记
└── references/             # 引用资料
```

## 相关资源

- → [[../../00-MOC/项目看板|项目看板 MOC]]
- → [[../../Templates/tpl-项目 README|项目 README 模板]]
- → [[../README|项目层根]]
- → [[../Planning/README|规划中项目]]
- → [[../Archive/README|已归档项目]]
- → [[../../00-MOC/KB 构建规范|KB 构建规范]] §三 笔记类型
