---
title: CrewAI Flow
aliases: [CrewAI Flow, Flow, CrewAI Flow Orchestration]
tags:
  - framework
  - topic/orchestration
  - topic/multi-agent
  - status/reviewed
created: 2026-06-10
updated: 2026-06-10
language: Python
version: ""
repo: https://github.com/crewAIInc/crewAI
license: MIT
---

# CrewAI Flow

> CrewAI 的声明式工作流编排功能。通过 `@start`、`@listen`、`@router` 装饰器定义事件驱动的执行流程，支持状态管理、条件路由、持久化和人机协作。

## 基本信息

| 项目 | 详情 |
|------|------|
| 官网 | https://docs.crewai.com/concepts/flows |
| GitHub | https://github.com/crewAIInc/crewAI |
| 语言 | Python |
| 所属框架 | CrewAI |
| 许可证 | MIT |
| 最低版本要求 | crewai >= 0.8.0 |

## 核心架构

Flow 是 CrewAI 的工作流编排层，位于 Crew 和 Task 之上，提供**声明式、事件驱动**的执行模型：

```
Flow 编排层
┌─────────────────────────────────────────────┐
│  @start() ──> @listen() ──> @router()       │
│      │            │            │             │
│      v            v            v             │
│  ┌───────┐   ┌───────┐   ┌─────────┐       │
│  │ Task  │   │ Crew  │   │ Branch  │       │
│  └───────┘   └───────┘   └─────────┘       │
│                    │                         │
│              ┌─────┴─────┐                   │
│              │   State   │                   │
│              │ (Pydantic │                   │
│              │  / dict)  │                   │
│              └───────────┘                   │
└─────────────────────────────────────────────┘
```

### 核心装饰器

| 装饰器 | 功能 | 说明 |
|--------|------|------|
| `@start()` | 入口点 | 标记 Flow 的起始方法，多个 `@start` 方法会并行执行 |
| `@listen()` | 事件监听 | 监听其他方法的输出，收到输出后触发执行 |
| `@router()` | 条件路由 | 根据返回值动态选择下游分支 |
| `@persist()` | 状态持久化 | 自动将状态保存到 SQLite 等存储后端 |
| `@human_feedback()` | 人机协作 | 暂停执行等待人工反馈 |

## 快速上手

### 安装

```bash
pip install crewai crewai-tools
```

### 最小示例

```python
from crewai.flow.flow import Flow, listen, start
from litellm import completion

class SimpleFlow(Flow):
    model = "gpt-4o-mini"

    @start()
    def generate_city(self):
        response = completion(
            model=self.model,
            messages=[{"role": "user", "content": "返回一个随机城市名称"}],
        )
        city = response["choices"][0]["message"]["content"]
        self.state["city"] = city
        return city

    @listen(generate_city)
    def generate_fact(self, city):
        response = completion(
            model=self.model,
            messages=[{"role": "user", "content": f"告诉我一个关于{city}的有趣事实"}],
        )
        fact = response["choices"][0]["message"]["content"]
        self.state["fact"] = fact
        return fact

# 执行 Flow
flow = SimpleFlow()
result = flow.kickoff()
print(result)
```

### 结构化状态管理

```python
from pydantic import BaseModel
from crewai.flow.flow import Flow, listen, start

class FlowState(BaseModel):
    counter: int = 0
    message: str = ""

class MyFlow(Flow[FlowState]):

    @start()
    def step_one(self):
        self.state.message = "Hello"
        self.state.counter += 1

    @listen(step_one)
    def step_two(self):
        self.state.message += " World"
        self.state.counter += 1
        return self.state.message

flow = MyFlow()
result = flow.kickoff()
print(f"Result: {result}")       # Hello World
print(f"Counter: {flow.state.counter}")  # 2
```

## 关键概念

### @start() 入口点

`@start()` 标记 Flow 的起始方法：

```python
# 无条件入口（Flow 启动时执行）
@start()
def begin(self):
    ...

# 条件入口（依赖某个方法或 router 标签完成后才执行）
@start("method_or_label")
def conditional_start(self):
    ...

# 多个 @start 方法会并行执行
@start()
def task_a(self):
    ...

@start()
def task_b(self):
    ...
```

### @listen() 事件监听

`@listen()` 建立方法间的依赖关系：

```python
# 方式1：直接引用方法
@listen(generate_city)
def process_city(self, city):
    ...

# 方式2：字符串引用
@listen("generate_city")
def process_city(self, city):
    ...

# 监听 router 的特定分支
@listen("success")
def handle_success(self):
    ...

@listen("failed")
def handle_failure(self):
    ...
```

### @router() 条件路由

`@router()` 实现动态分支：

```python
from crewai.flow.flow import Flow, listen, router, start

class RouterFlow(Flow):

    @start()
    def begin(self):
        self.state["score"] = evaluate_something()

    @router(begin)
    def decide(self):
        if self.state["score"] > 0.8:
            return "high"
        elif self.state["score"] > 0.5:
            return "medium"
        else:
            return "low"

    @listen("high")
    def handle_high(self):
        print("高分路径")

    @listen("medium")
    def handle_medium(self):
        print("中等路径")

    @listen("low")
    def handle_low(self):
        print("低分路径")
```

### or_() 和 and_() 组合逻辑

```python
from crewai.flow.flow import Flow, listen, or_, and_, start

class CombinatorFlow(Flow):

    @start()
    def task_a(self):
        return "A done"

    @start()
    def task_b(self):
        return "B done"

    # 任一完成即触发
    @listen(or_(task_a, task_b))
    def on_any_complete(self, result):
        print(f"某个任务完成: {result}")

    # 全部完成才触发
    @listen(and_(task_a, task_b))
    def on_all_complete(self):
        print("所有任务都完成了")
```

### @persist() 状态持久化

```python
from crewai.flow.flow import Flow, start, listen
from crewai.flow.persistence import persist
from pydantic import BaseModel

class CounterState(BaseModel):
    counter: int = 0

@persist  # 类级别持久化，所有方法状态自动保存
class CounterFlow(Flow[CounterState]):

    @start()
    def increment(self):
        self.state.counter += 1
        print(f"Counter: {self.state.counter}")

# 恢复之前的状态继续执行
flow = CounterFlow()
flow.kickoff(inputs={"id": "my-flow-id"})  # 恢复指定 ID 的状态

# Fork：基于已有状态创建新实例
flow2 = CounterFlow()
flow2.kickoff(restore_from_state_id=flow.state.id)
```

### @human_feedback() 人机协作

```python
from crewai.flow.flow import Flow, start, listen
from crewai.flow.human_feedback import human_feedback, HumanFeedbackResult

class ReviewFlow(Flow):

    @start()
    @human_feedback(
        message="请审核此内容是否可以发布？",
        emit=["approved", "rejected", "needs_revision"],
        llm="gpt-4o-mini",
        default_outcome="needs_revision",
    )
    def generate_content(self):
        return "生成的内容..."

    @listen("approved")
    def on_approved(self, result: HumanFeedbackResult):
        print(f"已批准: {result.feedback}")

    @listen("rejected")
    def on_rejected(self, result: HumanFeedbackResult):
        print(f"已拒绝: {result.feedback}")
```

### 嵌入 Agent 和 Crew

```python
from crewai import Agent, Crew, Task
from crewai.flow.flow import Flow, listen, start

class MultiCrewFlow(Flow):

    @start()
    def research_phase(self):
        researcher = Agent(
            role="研究员",
            goal="深入研究主题",
            backstory="你是资深研究分析师。"
        )
        task = Task(
            description=f"研究: {self.state['topic']}",
            agent=researcher,
            expected_output="详细研究报告"
        )
        crew = Crew(agents=[researcher], tasks=[task])
        result = crew.kickoff()
        self.state["research"] = result
        return result

    @listen(research_phase)
    def writing_phase(self, research):
        writer = Agent(
            role="撰稿人",
            goal="撰写技术文章",
            backstory="你是技术写作专家。"
        )
        task = Task(
            description=f"基于以下研究撰写文章:\n{research}",
            agent=writer,
            expected_output="技术文章"
        )
        crew = Crew(agents=[writer], tasks=[task])
        return crew.kickoff()
```

### 可视化 Flow

```python
flow = MyFlow()
flow.plot("my_flow_graph")  # 生成 HTML 可视化文件
```

## 典型使用场景

- **多阶段内容生产**：研究 -> 大纲 -> 撰写 -> 审校，每阶段可用不同 Crew
- **条件决策流**：根据分析结果选择不同处理路径（如情感分析后分流）
- **数据处理管道**：数据收集 -> 清洗 -> 分析 -> 报告，支持并行分支
- **人机协作流程**：AI 生成初稿 -> 人工审核 -> AI 修改 -> 最终发布
- **多 Crew 协调**：研究 Crew 和写作 Crew 通过 Flow 串联协作
- **持久化工作流**：长时间运行的任务支持断点续传和状态恢复

## 与同类对比

| 维度 | CrewAI Flow | LangGraph | Prefect/Dagster | Temporal |
|------|-------------|-----------|-----------------|----------|
| 定位 | AI 工作流编排 | AI 图编排 | 数据流水线 | 通用工作流 |
| 声明方式 | 装饰器 | 图定义 | 装饰器/DSL | 代码定义 |
| 状态管理 | Pydantic/dict | 图状态 | 参数传递 | 持久化状态 |
| 条件路由 | `@router` | 条件边 | 分支任务 | 信号/查询 |
| 持久化 | 内置 SQLite | 需外部 | 内置 | 内置 |
| 人机协作 | 内置 `@human_feedback` | 需自建 | 需自建 | 内置 |
| 学习曲线 | 低 | 中 | 中高 | 高 |

## 已知局限

- **调试复杂度**：装饰器魔法使执行流程不易追踪，需依赖 `plot()` 可视化
- **错误传播**：异步并行分支的错误处理机制不够完善
- **状态序列化**：复杂对象（如数据库连接）不能直接放入状态
- **持久化后端**：默认仅支持 SQLite，生产环境可能需要 Redis/PostgreSQL
- **测试难度**：Flow 的事件驱动特性使单元测试需要特殊处理
- **版本兼容**：部分功能（如 `@human_feedback`）需要较新版本

## 参考资料

- [CrewAI Flows 官方文档](https://docs.crewai.com/concepts/flows)
- [CrewAI GitHub](https://github.com/crewAIInc/crewAI)
- [CrewAI Flow 示例](https://github.com/crewAIInc/crewAI/tree/main/tests/flow)
- [CrewAI Tools](https://github.com/crewAIInc/crewAI-tools)
