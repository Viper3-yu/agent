---
title: KB 构建规范
aliases: [KB Schema, 知识库 schema]
tags:
  - MOC
  - meta/schema
  - status/reviewed
created: 2026-06-13
updated: 2026-06-13
source: [[../Clippings/llm-wiki]]
---

# KB 构建规范（Schema）

> 这是 Karpathy "LLM Wiki" 模式中定义的 **schema 文件**——告诉 Claudian 这个 vault 的结构、约定和工作流。
> 与老大共建、持续演化。任何规则变更先在这里改，再由 Claudian 后续会话遵守。

**理论依据**：[[../Clippings/llm-wiki|LLM Wiki 模式（Karpathy）]]

---

## 一、三层架构映射

Karpathy 模式的三层 + 我们 vault 的对应：

| Karpathy 层 | 职责 | 我们 vault 的位置 | 说明 |
|------------|------|-----------------|------|
| **Raw Sources** | 原始资料（不可变） | `Clippings/` | 老大剪藏的文章、论文、笔记 |
| **Wiki** | LLM 生成的 markdown 知识库 | `01-Concepts/` `02-Models/` `03-Prompt/` `04-Frameworks/` `05-Projects/` `06-Resources/` `00-MOC/` | 由 Claudian 维护，老大审阅 |
| **Schema** | 配置文件 + 工作流 | 本文件 `00-MOC/KB 构建规范.md` + `log.md` | 老大与 Claudian 共建 |

### 不属于 Wiki 的特殊位置

- **`Inbox/`** — 临时收件箱（老大可手动 drop 任何东西）
- **`Inbox/_drafts/`** — Claudian 草稿（待审阅状态）
- **`Templates/`** — Templater 模板
- **`.obsidian/`** — Obsidian 应用配置（不进 git）

---

## 二、三大操作（Operations）

### 1. Ingest（摄入新原始资料）

**触发**：老大往 `Clippings/` 或 `Inbox/` 放新文件，并要求处理。

**流程**：
```
1. Claudian 读 Clippings/<新文件> 全文
2. 提取关键信息（与老大讨论要点）
3. 写入 wiki 的对应类型笔记（用 tpl-* 模板）
4. 更新相关 MOC 和反向引用
5. 在 log.md 追加 `## [DATE] ingest | <标题>`
```

**特点**：单源可触发 5-15 个 wiki 页面的连锁更新。

### 2. Query（查询）

**触发**：老大提任何问题。

**流程**：
```
1. 读 00-MOC/* 找相关分类
2. 读相关笔记（带 wikilinks 串读）
3. 必要时查 Clippings/ 原始资料
4. 必要时做 Lint（健康检查）
5. 用 inline 引用回答（[[笔记名]] 可点击）
6. 重要回答可回写为 wiki 笔记（Ingest 子流程）
```

**约束**：
- 引用必须给出真实 wikilink
- 不确定时主动说明，避免幻觉
- 高价值回答应主动建议归档到 wiki

### 3. Lint（健康检查）

**触发**：老大要求，或 Claudian 在重大改动后主动提议。

**流程**：
```
1. 全 vault 扫描（Glob 找所有 .md）
2. 检查项：
   - 断链（wikilink 指向不存在的笔记）
   - 反向引用（MOC ↔ 笔记 ↔ 笔记）
   - frontmatter 完整性
   - status/seed 笔记（应升级）
   - 占位笔记（占位目录未启动）
   - 命名约定一致性
3. 生成诊断报告（Inbox/00-知识库诊断报告 YYYY-MM-DD.md）
4. 给老大优先级排序的修复建议
5. 修复后追加 log
```

---

## 三、笔记类型与模板

### 类型 → 目录 → 模板

| 类型 | 目录 | 模板 | 专属 frontmatter 字段 |
|------|------|------|---------------------|
| 概念 | `01-Concepts/<子分类>/` | `tpl-概念笔记.md` | — |
| 模型 | `02-Models/<厂商>/` | `tpl-模型笔记.md` | `provider`, `released`, `tier` |
| 框架 | `04-Frameworks/<分类>/` | `tpl-框架工具笔记.md` | `language`, `version`, `repo`, `license` |
| 协议 | `04-Frameworks/<分类>/` | `tpl-协议笔记.md` | `spec_url`, `maintainer`, `transport` |
| Agent 产品 | `04-Frameworks/Agent 产品/` | `tpl-Agent产品笔记.md` | `vendor`, `category`, `pricing`, `url` |
| 构建模式 | `01-Concepts/Agent 构建模式/` | `tpl-构建模式笔记.md` | `category`, `complexity`, `source` |
| 项目 | `05-Projects/<状态>/` | `tpl-项目 README.md` | `status` |
| 论文 | `06-Resources/Papers/` | `tpl-论文笔记.md` | `authors`, `year`, `venue`, `url` |
| 评估指标 | `04-Frameworks/Agent 可观测性/` | `tpl-评估笔记.md` | `category`, `url` |
| 微调方法 | `06-Resources/微调/` | `tpl-微调笔记.md` | `category`, `base_models`, `hardware` |

---

## 四、Frontmatter 规范

### 必填字段

```yaml
---
title: <笔记标题>                       # 用正式中文名
aliases: [<别名1>, <别名2>]              # 至少 1 个英文/简写别名
tags:
  - <类型>                              # concept / model / framework / 等
  - topic/<主题>                        # 可选，参考下方枚举
  - status/<状态>                       # 必填
  - meta/<元>                           # 仅 meta 类笔记
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

### Type Tag 枚举（互斥）

`concept` `model` `framework` `protocol` `prompt` `product` `paper` `project` `fine-tuning` `pattern` `evaluation` `MOC` `meta/...`

### Topic Tag（topic/*）

| Tag | 含义 | 当前使用 |
|-----|------|---------|
| `topic/agent` | Agent 系统 | ✅ |
| `topic/llm` | LLM 基础理论 | ✅ |
| `topic/rag` | RAG / 检索 | — |
| `topic/finetune` | 微调 | — |
| `topic/eval` | 评测 | — |
| `topic/safety` | 对齐与安全 | — |
| `topic/multimodal` | 多模态 | — |
| `topic/reasoning` | 推理 | — |
| `topic/planning` | 规划 | ✅ |
| `topic/memory` | 记忆 | — |
| `topic/coding` | 编程 Agent | — |
| `topic/prompt` | Prompt 工程 | — |

### Status Tag（status/*）

| Tag | 含义 | 触发 |
|-----|------|------|
| `status/seed` | 初始占位 | 仅占位笔记 |
| `status/draft` | 起草中 | Claudian 起草未审 |
| `status/reviewed` | 已审可用 | 老大点头通过 |
| `status/mature` | 已成熟 | 深填充 + 多链接 + 多次引用 |
| `status/archived` | 已归档 | 协议废弃、产品停服 |

---

## 五、笔记结构规范

### 概念笔记 7 段式（最小完整）

```markdown
# <标题>
> 一句话定位

## 核心思想        # What + Why
## 工作原理        # How（含公式/伪代码/图示）
## 优势与局限      # Trade-offs
## 相关概念        # wikilinks（≥3 个）
## 参考资料        # 论文/官方/博客
## 时间线          # 可选，适用协议/模型
```

### 协议笔记 9 段式（在 7 段基础上 +）

```markdown
## 设计动机        # 为什么需要这个协议
## 核心架构        # ASCII 或 mermaid 图
## 关键概念        # 清单
## 消息类型        # 表格
## 交互流程        # 时序
## 安全考量        # 鉴权/审计/限流
## 与其他协议的关系
## 代码示例        # 最小可运行
```

### 模型笔记 8 段式

```markdown
# <标题>
> 一句话定位
## 基本信息        # 表格（厂商/日期/上下文/定价）
## 核心能力        # bullet
## 版本链          # 前代/后继
## 横向对比        # 同档位模型
## benchmark       # 数字必须标注日期
## 使用场景
## 参考资料
## 已知局限        # 可选
```

---

## 六、链接规范

### Wikilink 风格（强约束）

```markdown
✅ [[../01-Concepts/Agent 架构/ReAct 架构|ReAct 架构]]   # 相对路径 + 显示文本
✅ [[MCP 原理]]                                            # 同目录可省略路径
❌ [[ReAct 架构]]                                          # 缺路径，Obsidian 唯一匹配可能错
❌ [ReAct](../path.md)                                     # 用 Markdown 链接代替 wikilink
```

### 必须的链接关系

| 笔记类型 | 必须链接到 |
|---------|----------|
| 模型 | 前代 + 后继 |
| 协议 | 同层其他协议 |
| 概念 | 上游 + 下游概念（≥3 个） |
| 框架 | 协议层 + 类似框架对比 |
| 项目 | 相关概念/资源 |

---

## 七、与 Claudian 的协作边界

### 流程总览

```
[1] 老大提需求
     ↓
[2] Claudian 拆任务（>3 步用 TaskCreate）
     ↓
[3] 起草 → Inbox/_drafts/<分类>/
     ↓
[4] 给交付清单 + 决策点
     ↓
[5] 老大点头 → 应用到正式位置
     ↓
[6] 清理草稿 + 追加 log
```

### Claudian 必须遵守的边界

1. **永不直接改正式位置**——除非老大明确"采纳/应用/继续"
2. **草稿先入 Inbox/_drafts/**——便于回滚
3. **批量写入前确认**——>3 个文件改动先列清单
4. **联网失败要标注**——⚠️ 标记未核实信息
5. **新原始资料先读**——动手前必读 Clippings/ 和 log.md 最近 5 条
6. **更新本 schema**——任何工作流变更先在这里改
7. **改动必走 feature branch**——>5 文件或 >200 行改动必须开分支，老大 review 后再 merge（详见下方「分支策略」）

### Claudian 可直接做的事

- 修复**纯技术性问题**（断链、frontmatter 字段缺失）
- 清理**已应用的旧草稿**
- 读取任何文件
- 在 Inbox 创建任何文件
- 追加 log.md

### Claudian 必须先读再答的问题

- "参考目前这篇笔记"——必须先找这篇笔记
- "基于 X 优化 Y"——必须先读 X
- 任何引用具体外部资料的问题

### 分支策略（2026-06-13 新增）

**核心规则**：`main` 分支**不可直改直推**。所有改动走 feature branch。

| 改动规模 | 走法 |
|---------|------|
| **< 5 文件 且 < 200 行** | 可在 main 直改直推（schema 等小调整） |
| **≥ 5 文件 或 ≥ 200 行** | **必须开 feature branch** |

#### 分支命名

`<type>/<scope>-<YYYY-MM-DD>`，例：
- `chore/workflow-2026-06-13`
- `fix/obsidian-graph-2026-06-13`
- `feat/concept-rlhf-2026-06-13`

#### 工作流

```bash
# 1. 从 main 切出
git checkout main
git pull origin main
git checkout -b <branch-name>

# 2. 改 + commit（每个逻辑改动一个 commit）
git add -A
git commit -m "type(scope): 描述"

# 3. push feature branch
git push -u origin <branch-name>

# 4. 老大在 GitHub 远端 review
#    https://github.com/Viper3-yu/agent/branches

# 5. 老大点头后，merge
git checkout main
git merge --no-ff <branch-name>
git push origin main

# 6. 清理
git branch -d <branch-name>
git push origin --delete <branch-name>
```

#### 老大可选项：GitHub branch protection

- 在 `https://github.com/Viper3-yu/agent/settings/branches` 设 main 为 protected
- 勾 "Require pull request before merging"
- 勾 "Require approvals"（单人 vault 可设 0 = 自己 approve）
- 这样 Claudian 即使忘了开分支，push 也会被 GitHub 拒

#### 教训（2026-06-13 第一次跑批量脚本踩坑）

> 写脚本批量改 frontmatter 时，**必须遍历所有行，不能 `lines[:fm_end+1]`**。
> 我第一次的脚本写了 `new_lines = []; for line in lines[:fm_end+1]: ...`，
> 漏了 body 的 9204 行。143 篇笔记 body 全没了，幸在 feature branch 里没污染 main。

---

## 八、工具栈

### Obsidian 插件（推荐）

| 插件 | 用途 | 状态 |
|------|------|------|
| **Templater** | 模板填充 | ✅ 已有 |
| **Dataview** | frontmatter 查询 | 待评估 |
| **obsidian-git** | 自动 commit + push | 待安装 |
| **obsidian-diff** | 内嵌看 diff | 待安装 |
| **Marp** | 生成幻灯片 | 按需 |

### CLI 工具（推荐）

| 工具 | 用途 | 来源 |
|------|------|------|
| **qmd** | 本地混合 BM25/向量检索 + LLM reranking | Karpathy 推荐（[GitHub](https://github.com/tobi/qmd)）|
| **git** | 版本管理 | 已建议 |

### 内容收集工具

- **Obsidian Web Clipper**（浏览器扩展）— 把 web 文章转 markdown 到 `Clippings/`

---

## 九、Git 版本管理

**强建议启用**（详见 Karpathy 文章末尾"The wiki is just a git repo of markdown files"）。

### `.gitignore` 模板（实际使用版，2026-06-13 落地）

**重要调整**：`.obsidian/plugins/` 是 41M（插件二进制）、`.obsidian/themes/` 是 1.1M，**全部排除**；只追踪配置 JSON，让新机器从 `core-plugins.json` / `community-plugins.json` 知道要装哪些插件，再从 Obsidian Community 重装。

```gitignore
# ===== Obsidian：追踪配置，忽略代码/缓存 =====
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/plugins/
.obsidian/themes/
.obsidian/cache/
.obsidian/plugins/*/data.json   # 兼容未来

# ===== 系统文件 =====
.DS_Store
Thumbs.db
desktop.ini
$RECYCLE.BIN/

# ===== 编辑器临时文件 =====
*.swp *.swo *.tmp *.bak *~
.vscode/ .idea/

# ===== Claudian 临时 =====
# Inbox/_drafts/  # 已应用的草稿由我清理；如需审计可取消注释
```

**保留跟踪的 .obsidian 文件**（新机器恢复必需）：
- `app.json` / `appearance.json` / `hotkeys.json` / `types.json` / `graph.json`
- `core-plugins.json` / `community-plugins.json`（插件列表）

### Commit 约定

```
<类型>(<范围>): <描述>

类型: feat / fix / refactor / docs / meta / chore / lint
范围: 笔记路径（01-Concepts / 02-Models / MOC / schema / log 等）
描述: 一句话

示例：
  feat(01-Concepts): 新增 Tokenization/Embedding/MoE/采样策略/量化
  fix(MOC): 修复 LLM 模型地图 10 处断链
  meta(schema): 用 Karpathy 模式重写 KB 构建规范
  chore(log): 新建 log.md
```

### 实施步骤

```bash
cd ~/obsidian1
git init -b main
# 写入上面的 .gitignore
git add . && git commit -m "chore: 初始化 vault 快照（197 篇笔记）"
# 创建私有远程仓库后（见下）
git remote add origin <your-private-repo-url>
git push -u origin main
# 安装 obsidian-git 插件，配置 5 分钟自动 commit
```

**远程仓库选项**：
- **GitHub 私有** — 全球访问、生态最好，Push 需科学上网
- **Gitee 私有** — 国内速度快，免费 5 人仓库
- **自建 Gitea/Forgejo** — 完全私有，最稳

---

## 十、维护节奏

### 老大触发 schema 更新

每当：
- 想改笔记结构
- 引入新类型笔记
- 改 frontmatter 规范
- 改 Claudian 协作流程
- 添加新工具

→ 更新本文件，Claudian 下次会话读取时遵守。

### Claudian 触发 log 追加

每次主要操作完成后：
- 追加 `log.md`
- 引用 schema 章节号（如有元规则变更）

---

## 附录 A：本 schema 的演化历史

| 日期 | 变更 | 触发 |
|------|------|------|
| 2026-06-13 | 首次创建（凭 Claudian 直觉） | 老大要求优化 KB 结构 |
| 2026-06-13 | 重写对齐 Karpathy "LLM Wiki" 模式 | 老大指出应参考 `Clippings/llm-wiki.md` |

---

— Claudian 起草，2026-06-13
- v1：直觉版
- v2：Karpathy 模式对齐版（本版）