---
title: LangGraph 实战，3 步搭建 Agent 彻底告别手搓痛苦
description: 本文面向 AI 开发初学者与刚入门工程师，聚焦复杂 Agent 开发的核心痛点 —— 手搓代码时流程控制混乱、状态管理繁琐、工具集成低效，以开发者从 “手搓党” 到 LangGraph “真香” 的真实经历为线索，系统拆解 LangGraph 的核心价值与实操方法
date: 2025-10-25
lastmod: 2025-10-25
author: Sherry
avatar: https://s3.aiedu.qzz.io/website/2025/10/c9636914b0782ae3f9269b50dcb88659.png
tags:
  - 技术
  - LangChain
categories:
  - 编程
  - 知识库
series: 开发指南
image: https://s3.aiedu.qzz.io/website/2025/11/5e4426c0ca882f889d025054b15ea300.png
---

在 LangGraph 出现之前，多工具协同 Agent 的开发堪称 “踩坑重灾区”：不仅要手动编写工具调用判断、结果处理、循环终止等胶水代码，逻辑冗余且上线后频繁出现 “漏处理工具结果” 等功能性 bug；用全局变量存储会话状态时，多用户并发场景下极易引发上下文覆盖问题，故障排查往往要耗费大量时间定位根因；更棘手的是新增单一工具（如 “物流查询”），竟需同步修改 5 处以上关联逻辑（判断条件、参数传递、结果拼接），稍有不慎就会引发其他流程的连锁故障。

不少开发者初期都会像我一样，误判框架 “抽象冗余”，执着于手搓代码，总觉得 “几行 Python 就能搞定复杂场景”，却忽视了复杂 Agent 开发的潜在难点，最终陷入被动开发的困境。直到邂逅 LangGraph，才彻底颠覆了这种认知 —— 原来高效开发 Agent 可以如此轻松！

很多开发者初期对框架的抵触情绪，其实都源于相似的顾虑。下面结合我曾经的困惑，聊聊 LangGraph 是如何逐一破解这些难题的。

### 1.1 开发者对框架的 3 大核心顾虑

|顾虑点|具体场景|手搓代码的 “伪优势”|
|---|---|---|
|抽象冗余|调用 LLM 只需简单的 llm.invoke ()，框架却要配置 Chain/Agent 等一系列组件|代码量少、直观易懂，无需记忆复杂的框架概念|
|灵活性不足|框架固定模板难以定制，比如想在工具调用前增加权限校验，需修改整个链路|逻辑全由自己掌控，可按需自由调整|
|学习成本高|需花费时间学习框架的 API 和核心概念（如 LangChain 的 Memory/Retriever）|依托现有 Python 基础即可开发，无需学习新内容|

### 1.2 LangGraph 的破局之道

当开发者在复杂 Agent 开发中踩坑后就会发现：手搓代码的 “灵活”，在复杂流程下会演变成 “混乱”；而框架的 “抽象”，实则是对重复工作的高效封装。

作为 LangChain 生态的进阶框架，LangGraph 并非简单的 “工具套件”，而是 Agent 的 “流程蓝图”。它通过**图结构**设计，精准解决了手搓代码的 3 大痛点：

|手搓代码痛点|LangGraph 解决方案|实际开发体验|
|---|---|---|
|流程控制混乱（分支 / 循环逻辑难编写）|以 “节点 + 边” 定义流程，分支通过条件边实现，循环依靠节点间闭环完成，无需手写 if-else/while 语句|新增工具调用循环时，仅需添加一条 “工具节点→LLM 节点” 的边，无需修改核心逻辑|
|状态管理复杂（会话 / 工具结果易丢失）|全局 State+Checkpointer 自动存储状态，所有节点共享数据，无需手动传递上下文|多轮对话无需每次携带历史消息，Checkpointer 可自动从数据库读取之前的状态|
|工具集成繁琐（参数解析 / 结果处理耗时）|内置 ToolNode 自动解析 LLM 的工具调用指令，省去 “提取工具名 + 参数” 的冗余代码|新增工具只需添加 @tool 装饰器，框架会自动处理后续调用逻辑|

## 二、3 分钟吃透 LangGraph 核心概念：图结构 Agent 入门

提到 “图结构”，新手无需畏惧。其实本质就是将 Agent 的工作拆解为 “最小功能单元（节点）”，明确这些单元的执行顺序（边），再搭配一个 “全局数据桶（State）”，就能构建出高效的 Agent 系统。

### 2.1 核心三要素：State、Node、Edge（必学基础）

这三个要素是 LangGraph 的基石，理解它们后，后续代码开发会事半功倍。

#### 2.1.1 State：Agent 的 “全局数据桶”

- **核心作用**：存储 Agent 运行时的所有数据，包括对话历史、工具结果、用户配置等，所有节点均可对其进行读写操作。
- **本质属性**：一个带类型约束的字典（通过 TypedDict 定义），新手可直接参考模板编写。
- **关键细节**：借助 add_messages 注解能自动追加对话历史，有效避免手搓代码时 “覆盖历史消息” 的低级错误。

python

运行

```python
# 新手可直接复用的State定义模板
from typing_extensions import TypedDict
from typing import Annotated
from langgraph.graph.message import add_messages

class AgentState(TypedDict):
    # 对话历史：add_messages自动处理消息追加，无需手动执行list.append()
    messages: Annotated[list, add_messages]
    # 可选：添加迭代计数器（用于防止无限循环）
    iteration_count: int  # 后续踩坑指南会具体用到
```

#### 2.1.2 Node：Agent 的 “功能单元”

- **核心作用**：封装具体业务逻辑（如调用 LLM、执行工具、处理数据），输入为 State，输出为更新后的 State。
- **常见类型**：
    1. LLM 节点：调用大模型生成回答或工具指令（例如 “判断是否需要查询天气”）；
    2. 工具节点：执行外部工具（例如调用搜索接口、查询数据库）；
    3. 决策节点：判断流程下一步的执行节点（与条件边配合使用）。

python

运行

```python
# 示例：LLM节点（新手可直接复用）
from langchain_openai import ChatOpenAI

# 初始化LLM（可替换为讯飞、百度等其他模型）
llm = ChatOpenAI(model="gpt-4o-mini", api_key="你的密钥")

def llm_node(state: AgentState):
    # 1. 从State中读取对话历史
    history = state["messages"]
    # 2. 调用LLM生成结果
    result = llm.invoke(history)
    # 3. 返回更新后的State（仅修改messages和计数器）
    return {
        "messages": [result],  # add_messages会自动完成消息追加
        "iteration_count": state["iteration_count"] + 1  # 迭代计数器加1
    }
```

#### 2.1.3 Edge：Agent 的 “流程导航”

- **核心作用**：定义节点之间的执行顺序，主要分为两种类型：

|边类型|核心作用|适用场景|
|---|---|---|
|普通边（Add Edge）|固定跳转逻辑，例如 “START→LLM 节点”|无需判断的线性流程（如简单聊天机器人）|
|条件边（Add Conditional Edges）|根据 State 数据动态调整跳转路径|分支 / 循环场景（如 “LLM 需调用工具则跳转至工具节点，否则结束流程”）|

python

运行

```python
# 示例：条件边的路由函数（判断是否需要调用工具）
def tool_route(state: AgentState) -> str:
    # 1. 读取LLM的最新输出结果
    last_msg = state["messages"][-1]
    # 2. 防止无限循环：迭代次数超过5次则强制结束流程
    if state["iteration_count"] >= 5:
        return "__end__"  # LangGraph内置的“结束节点”标识
    # 3. 判断LLM是否存在工具调用需求
    if last_msg.tool_calls:  # 当LLM输出包含tool_calls字段时，说明需要调用工具
        return "tool_node"  # 跳转至工具节点
    return "__end__"  # 无需调用工具则结束流程
```

### 2.2 LangGraph vs 手搓代码：核心优势对比

|能力维度|手搓代码|LangGraph|
|---|---|---|
|流程控制|依赖 if-else/while 语句，复杂流程下代码臃肿杂乱|图结构可视化呈现，分支 / 循环通过 “节点 + 边” 实现，修改流程无需改动核心逻辑|
|状态管理|手动传递上下文或存储至数据库，易出现漏传、数据冲突问题|全局 State+Checkpointer 自动管理状态，多用户通过 thread_id 实现隔离|
|工具集成|需手动编写提示词引导 LLM 输出工具格式，并自行解析结果|bind_tools 自动生成提示词，ToolNode 负责解析执行，全程无需手动干预|
|调试效率|依赖 print 日志排查问题，流程处于 “黑箱” 状态|支持生成 Mermaid 可视化流程图，通过 Time Travel 功能回溯历史状态|

## 三、实操指南：3 步搭建可落地的 Agent（附完整代码）

新手建议按 “基础聊天→工具集成→记忆功能” 的顺序逐步推进，每一步都能独立运行，快速建立开发信心。首先完成环境准备：

bash

```bash
# 安装依赖（新手可直接复制执行）
pip install langgraph langchain-openai typing-extensions sqlite3 python-dotenv
```

### 3.1 第一步：构建基础聊天机器人（理解图结构）

目标：实现 “用户发消息→LLM 回消息” 的线性流程，掌握 LangGraph 的基础用法。

#### 3.1.1 完整代码

python

运行

```python
from typing_extensions import TypedDict
from typing import Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv  # 用dotenv管理密钥，避免硬编码
import os

# 1. 加载环境变量（新手需在项目根目录创建.env文件，写入OPENAI_API_KEY=你的密钥）
load_dotenv()
api_key = os.getenv("OPENAI_API_KEY")

# 2. 定义State（存储对话历史和迭代计数器）
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    iteration_count: int

# 3. 定义LLM节点
llm = ChatOpenAI(model="gpt-4o-mini", api_key=api_key)
def llm_node(state: AgentState):
    history = state["messages"]
    result = llm.invoke(history)
    return {
        "messages": [result],
        "iteration_count": state["iteration_count"] + 1
    }

# 4. 构建图结构
graph = StateGraph(AgentState)
# 添加节点：为节点命名并绑定对应函数
graph.add_node("llm", llm_node)
# 定义执行路径：START→llm节点→END
graph.add_edge(START, "llm")
graph.add_edge("llm", END)

# 5. 编译图结构（生成可执行的Agent）
agent = graph.compile()

# 6. 测试运行
if __name__ == "__main__":
    # 初始State：对话历史为空，迭代计数器设为0
    initial_state = {
        "messages": [{"role": "user", "content": "介绍下你自己"}],
        "iteration_count": 0
    }
    # 调用Agent获取结果
    result = agent.invoke(initial_state)
    # 打印最终回复（取最后一条消息）
    print("Agent回复：", result["messages"][-1].content)
    # 生成可视化流程图（保存为chat_graph.png，便于直观理解）
    agent.get_graph().draw_mermaid_png(output_file_path="chat_graph.png")
```

#### 3.1.2 关键步骤解析

- 运行代码后会生成 chat_graph.png 文件，打开可看到 “START→llm→END” 的线性流程，帮助新手直观理解图结构；
- 务必使用 dotenv 管理密钥，避免将密钥硬编码在代码中，防止泄露。

### 3.2 第二步：集成工具调用（LangGraph 核心场景）

目标：实现 “LLM 判断工具调用需求→执行工具→基于工具结果生成回答” 的闭环流程。这是手搓代码最繁琐的场景，也是 LangGraph 的核心优势所在。

#### 3.2.1 完整代码（基于第一步扩展）

python

运行

```python
from typing_extensions import TypedDict
from typing import Annotated
from langgraph.graph import StateGraph, START, END, ToolNode
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI
from langchain.tools import tool
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv("OPENAI_API_KEY")

# 1. 定义工具（新手可替换为真实工具，如天气API、搜索接口等）
class MyTools:
    @tool  # 必须添加@tool装饰器，框架才能识别工具
    def google_search(self, query: str) -> str:
        """用于获取互联网最新信息，例如新闻、天气、实时数据等"""
        # 此处为模拟结果，真实场景需替换为Google Search API调用逻辑
        return f"【搜索结果】关于'{query}'的信息：2025年诺贝尔物理学奖得主是XX团队（模拟数据）"

    @tool
    def code_check(self, language: str, code: str) -> str:
        """检查代码语法错误，支持Python/Java/JavaScript等语言"""
        # 此处为模拟结果，真实场景可集成pylint等工具
        return f"【代码检查结果】{language}代码无语法错误，逻辑正常（模拟数据）"

# 2. 定义State（与第一步保持一致，无需修改）
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    iteration_count: int

# 3. 初始化LLM并绑定工具
llm = ChatOpenAI(model="gpt-4o-mini", api_key=api_key)
tools = [MyTools().google_search, MyTools().code_check]  # 实例化工具列表
llm_with_tools = llm.bind_tools(tools)  # 为LLM绑定工具，自动生成工具调用提示词

# 4. 定义节点
# LLM节点：判断是否需要调用工具
def llm_node(state: AgentState):
    history = state["messages"]
    result = llm_with_tools.invoke(history)  # 使用绑定工具的LLM
    return {
        "messages": [result],
        "iteration_count": state["iteration_count"] + 1
    }

# 工具节点：LangGraph内置ToolNode，自动执行工具调用
tool_node = ToolNode(tools)

# 5. 定义条件边（判断流程走向）
def tool_route(state: AgentState) -> str:
    # 防止无限循环
    if state["iteration_count"] >= 5:
        return "__end__"
    # 判断LLM是否存在工具调用需求
    last_msg = state["messages"][-1]
    return "tool_node" if last_msg.tool_calls else "__end__"

# 6. 构建图结构
graph = StateGraph(AgentState)
# 添加节点（LLM节点+工具节点）
graph.add_node("llm", llm_node)
graph.add_node("tool_node", tool_node)
# 定义基础执行路径：START→llm节点
graph.add_edge(START, "llm")
# 定义条件边：llm节点根据判断结果跳转至工具节点或结束流程
graph.add_conditional_edges(
    source="llm",  # 源节点：llm节点
    condition=tool_route,  # 路由判断函数
    mapping={"tool_node": "tool_node", "__end__": "__end__"}  # 路由映射关系
)
# 工具节点执行完成后，返回至llm节点（形成闭环流程）
graph.add_edge("tool_node", "llm")

# 7. 编译并测试
agent = graph.compile()
if __name__ == "__main__":
    # 测试工具调用场景
    tool_test_state = {
        "messages": [{"role": "user", "content": "2025年诺贝尔物理学奖得主是谁？需要搜索"}],
        "iteration_count": 0
    }
    result1 = agent.invoke(tool_test_state)
    print("工具调用场景回复：", result1["messages"][-1].content)

    # 测试无需工具调用场景
    no_tool_test_state = {
        "messages": [{"role": "user", "content": "介绍下你自己"}],
        "iteration_count": 0
    }
    result2 = agent.invoke(no_tool_test_state)
    print("\n无工具场景回复：", result2["messages"][-1].content)

    # 生成可视化流程图（查看闭环逻辑）
    agent.get_graph().draw_mermaid_png(output_file_path="tool_agent_graph.png")
```

#### 3.2.2 核心优势体验

- 无需手动编写工具解析代码：ToolNode 会自动识别 LLM 输出的 tool_calls 字段，完成工具调用；
- 循环流程极简实现：通过 “tool_node→llm” 的边，即可让 Agent 基于工具结果继续思考，无需手写 while 循环；
- 工具扩展成本低：新增工具时，仅需在 MyTools 类中添加带 @tool 装饰器的方法，无需修改其他逻辑。

### 3.3 第三步：添加记忆功能（解决跨轮对话）

目标：让 Agent 具备跨轮对话记忆能力（例如用户说 “我叫小明”，后续询问 “我是谁” 时能准确应答）。手搓代码需手动对接数据库，而 LangGraph 通过 Checkpointer 可自动实现记忆管理。

#### 3.3.1 完整代码（实现会话内短期记忆）

python

运行

```python
from typing_extensions import TypedDict
from typing import Annotated
from langgraph.graph import StateGraph, START, END, ToolNode
from langgraph.graph.message import add_messages
from langgraph.checkpoint.sqlite import SqliteSaver  # 基于SQLite存储状态
from langchain_openai import ChatOpenAI
from langchain.tools import tool
from dotenv import load_dotenv
import os
import sqlite3

load_dotenv()
api_key = os.getenv("OPENAI_API_KEY")

# 1. 工具定义（与第二步保持一致，无需修改）
class MyTools:
    @tool
    def google_search(self, query: str) -> str:
        return f"【搜索结果】关于'{query}'的信息：2025年诺贝尔物理学奖得主是XX团队（模拟数据）"

# 2. State定义（与之前保持一致）
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    iteration_count: int

# 3. LLM和节点定义（与第二步保持一致）
llm = ChatOpenAI(model="gpt-4o-mini", api_key=api_key)
tools = [MyTools().google_search]
llm_with_tools = llm.bind_tools(tools)

def llm_node(state: AgentState):
    history = state["messages"]
    result = llm_with_tools.invoke(history)
    return {
        "messages": [result],
        "iteration_count": state["iteration_count"] + 1
    }

tool_node = ToolNode(tools)

def tool_route(state: AgentState) -> str:
    if state["iteration_count"] >= 5:
        return "__end__"
    return "tool_node" if state["messages"][-1].tool_calls else "__end__"

# 4. 初始化短期记忆（基于SQLite的Checkpointer）
# 创建SQLite数据库连接（用于存储状态数据）
conn = sqlite3.connect("agent_memory.db", check_same_thread=False)
# 初始化Checkpointer：通过SQLite持久化状态
memory = SqliteSaver(conn)

# 5. 构建图结构（与第二步保持一致）
graph = StateGraph(AgentState)
graph.add_node("llm", llm_node)
graph.add_node("tool_node", tool_node)
graph.add_edge(START, "llm")
graph.add_conditional_edges("llm", tool_route, {"tool_node": "tool_node", "__end__": "__end__"})
graph.add_edge("tool_node", "llm")

# 6. 编译时传入Checkpointer（启用记忆功能）
agent = graph.compile(checkpointer=memory)

# 7. 测试跨轮对话
if __name__ == "__main__":
    # 配置用户标识：通过thread_id隔离不同用户（例如user_001对应小明，user_002对应小红）
    user_config = {"configurable": {"thread_id": "user_001"}}
    
    # 第一轮对话：告知Agent用户姓名
    round1_state = {
        "messages": [{"role": "user", "content": "我叫小明，你记住了吗？"}],
        "iteration_count": 0
    }
    result1 = agent.invoke(round1_state, config=user_config)
    print("第一轮回复：", result1["messages"][-1].content)
    
    # 第二轮对话：询问用户姓名（无需携带历史消息，Checkpointer自动读取）
    round2_state = {
        "messages": [{"role": "user", "content": "我是谁？"}],
        "iteration_count": 0  # 计数器会从历史状态续接，此处设0不影响
    }
    result2 = agent.invoke(round2_state, config=user_config)
    print("\n第二轮回复：", result2["messages"][-1].content)
```

#### 3.3.2 长期记忆实现方案（生产级场景）

若需实现 “跨会话记忆”（例如用户间隔 3 天再次访问，Agent 仍能记住其偏好），LangGraph 内置的 InMemoryStore 无法满足需求（服务重启后数据丢失），建议采用以下方案：

- 短期记忆：分布式场景使用 RedisSaver，单机场景使用 SqliteSaver；
- 长期记忆：对接外部 RAG 系统（如 Pinecone 向量数据库），在工具函数中调用 RAG 接口获取数据（例如编写 get_user_preference 工具，从 RAG 系统查询用户偏好）。

## 四、落地踩坑指南：4 个高频问题解决方案

以下是实操过程中遇到的典型问题及解决方案，新手避开这些坑能大幅提升开发效率：

### 4.1 坑 1：内置 Store 无法支撑长期记忆需求

- **问题描述**：使用 InMemoryStore 存储长期数据时，服务重启后数据会全部丢失；
- **解决方案**：生产环境采用 “向量数据库（Pinecone/Milvus）+ LangChain Retriever” 的组合，在工具中调用 RAG 接口获取长期数据，避免依赖 LangGraph 内置 Store。

### 4.2 坑 2：非 OpenAI 模型适配困难

- **问题描述**：使用讯飞、百度等国内模型或 Llama 3 等开源模型时，llm.bind_tools 功能无法正常使用；
- **解决方案**：自定义 LLM 类（继承 LangChain 的 BaseChatModel），重写_generate 方法，确保输出格式包含 tool_calls 字段（与 OpenAI 输出格式兼容）。

### 4.3 坑 3：工具调用陷入无限循环

- **问题描述**：LLM 反复触发工具调用，无法正常结束流程；
- **解决方案**：在 State 中添加 iteration_count 字段，在条件边的路由函数中设置阈值（如迭代超过 5 次），强制结束流程（参考 3.2 节的 tool_route 函数）。

### 4.4 坑 4：State 数据冗余导致性能下降

- **问题描述**：将工具调用中间日志等临时数据存入 State，导致 LLM 上下文过长，执行效率降低；
- **解决方案**：State 仅存储 “必须共享的数据”（对话历史、工具结果、核心配置），临时数据在节点函数中生成后直接使用，无需存入 State。

## 五、框架选型指南：LangGraph vs 主流 Agent 框架

新手在选择框架时容易纠结，以下通过对比不同框架的核心特性，帮助大家按需选型：

|框架|核心架构|核心优势|主要劣势|适用场景|
|---|---|---|---|---|
|LangGraph|有向循环图|流程控制能力强、状态管理高效、调试便捷|学习成本中等（需理解图结构概念）|复杂 Agent 开发（多工具协同、长会话、分支循环场景）|
|LangChain|线性流水线|组件丰富、轻量易用、入门门槛低|复杂流程需手动编写控制逻辑|简单场景（单工具调用、快速原型开发）|
|AutoGen|角色化对话网络|多智能体协作能力突出|流程结构化弱、调试难度大|模拟团队协作（如方案编写、代码评审）|
|LlamaIndex|索引驱动 RAG|非结构化数据检索能力强|多步骤决策能力薄弱|知识库问答（如企业文档查询）|

### 选型建议

- 开发复杂 Agent（如智能客服、多功能助手）：优先选择 LangGraph；
- 快速实现简单工具调用功能：使用 LangChain 高效开发；
- 需多智能体协同工作：选用 AutoGen；
- 专注于文档问答场景：LlamaIndex 是更优选择。

## 六、总结：LangGraph 的核心价值

从抵触框架到离不开，LangGraph 让我深刻体会到：优秀的框架并非增加开发负担，而是通过封装重复、复杂的工作，让开发者能专注于核心业务逻辑。

它的核心价值可概括为 3 点：

1. **流程可视化**：用图结构替代繁琐的条件判断语句，修改流程无需改动核心代码；
2. **状态自动化**：自动完成上下文传递和状态存储，省去手动对接数据库的冗余工作；
3. **工具标准化**：新增工具仅需添加装饰器，框架自动处理解析和调用逻辑。

新手无需畏惧 “图结构” 的概念，跟着本文的 3 步实操流程动手实践，就能快速掌握 LangGraph 的使用技巧。相比手搓代码，它能节省至少 50% 的开发时间，可视化调试功能也能大幅提升问题排查效率。