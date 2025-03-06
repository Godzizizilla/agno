# Agent 记忆系统

Agent的记忆系统允许它存储和检索过去的交互历史，维护会话状态，并在多次交互中保持上下文连贯性。

## 记忆系统架构

```mermaid
graph TD
    A[Agent记忆系统] --> B[会话历史]
    A --> C[会话状态]
    A --> D[持久化存储]
    
    B --> B1[消息历史]
    B --> B2[工具调用历史]
    
    C --> C1[状态变量]
    C --> C2[上下文信息]
    
    D --> D1[数据库存储]
    D --> D2[文件存储]
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
```

## 记忆加载流程

```mermaid
flowchart TD
    A[Agent初始化] --> B{是否提供session_id?}
    
    B -->|是| C[尝试加载现有会话]
    B -->|否| D[创建新会话]
    
    C --> E{会话是否存在?}
    E -->|是| F[加载会话历史]
    E -->|否| D
    
    F --> G[加载会话状态]
    D --> H[初始化空会话]
    
    G --> I[准备记忆系统]
    H --> I
    
    I --> J[完成记忆初始化]
```

## 记忆使用流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant Agent as Agent
    participant Memory as 记忆系统
    participant Storage as 存储系统
    
    User->>Agent: 发送消息
    activate Agent
    
    Agent->>Memory: 加载会话历史
    activate Memory
    Memory-->>Agent: 返回历史消息
    deactivate Memory
    
    Agent->>Agent: 处理用户请求
    
    Agent->>Memory: 添加新消息到历史
    activate Memory
    Memory-->>Agent: 确认添加
    deactivate Memory
    
    opt 启用存储
        Agent->>Storage: 保存会话到存储
        activate Storage
        Storage-->>Agent: 确认保存
        deactivate Storage
    end
    
    Agent-->>User: 返回响应
    deactivate Agent
```

## 记忆数据结构

```mermaid
classDiagram
    class AgentMemory {
        +Dict messages
        +Dict state
        +add_message(message)
        +get_messages()
        +set_state(key, value)
        +get_state(key)
    }
    
    class AgentRun {
        +String run_id
        +List messages
        +Dict metrics
        +Dict extra_data
    }
    
    class AgentSession {
        +String session_id
        +String session_name
        +List runs
        +Dict state
        +add_run(run)
        +get_last_run()
    }
    
    AgentMemory "1" -- "1" AgentSession
    AgentSession "1" o-- "many" AgentRun
```

## 会话状态管理

```mermaid
flowchart LR
    A[会话状态] --> B[设置状态变量]
    A --> C[获取状态变量]
    A --> D[更新状态变量]
    
    B --> E[session_state字典]
    C --> E
    D --> E
    
    E --> F[在消息中使用状态]
    E --> G[在工具中使用状态]
    E --> H[持久化状态]
```

## 记忆系统代码示例

```python
from agno.agent import Agent
from agno.memory.agent import AgentMemory
from agno.storage.agent.sqlite import SQLiteAgentStorage

# 创建带有持久化存储的Agent
agent = Agent(
    name="记忆助手",
    # 配置存储
    storage=SQLiteAgentStorage(db_path="agent_sessions.db"),
    # 启用会话状态
    session_state={"counter": 0},
    # 在消息中使用状态变量
    add_state_in_messages=True
)

# 首次运行
response = agent.run("你好，这是我们第一次对话")

# 更新状态
agent.session_state["counter"] = 1
agent.session_state["user_name"] = "张三"

# 再次运行，会保持会话上下文
response = agent.run("你还记得我们之前聊了什么吗?")

# 保存会话ID以便未来恢复
session_id = agent.session_id

# 稍后恢复会话
restored_agent = Agent(
    name="记忆助手",
    storage=SQLiteAgentStorage(db_path="agent_sessions.db"),
    session_id=session_id  # 提供之前的会话ID
)

# 继续之前的对话
response = restored_agent.run("继续我们之前的对话")
```

记忆系统是Agent长期交互的基础，它使Agent能够记住过去的对话，维护状态，并在多次交互中提供连贯的体验。 