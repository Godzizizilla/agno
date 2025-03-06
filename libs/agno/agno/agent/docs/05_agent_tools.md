# Agent 工具使用

Agent的工具使用功能允许它调用外部函数来执行各种任务，极大地扩展了Agent的能力范围。工具可以是任何Python函数，从简单的计算到复杂的API调用。

## 工具调用流程

```mermaid
flowchart TD
    A[Agent接收用户输入] --> B[调用模型]
    B --> C{模型是否请求工具?}
    
    C -->|否| D[返回文本响应]
    C -->|是| E[解析工具调用请求]
    
    E --> F[查找工具定义]
    F --> G{工具是否存在?}
    
    G -->|否| H[返回错误]
    G -->|是| I[验证参数]
    
    I --> J{参数是否有效?}
    J -->|否| K[返回参数错误]
    J -->|是| L[执行工具函数]
    
    L --> M[获取工具结果]
    M --> N[将结果返回给模型]
    N --> O[模型生成最终响应]
```

## 工具定义和注册

```mermaid
classDiagram
    class Function {
        +String name
        +String description
        +Dict parameters
        +Callable function
        +Dict schema
    }
    
    class Toolkit {
        +String name
        +List~Function~ functions
        +add_function(function)
        +get_schemas()
    }
    
    class Agent {
        +List~Union[Function, Toolkit]~ tools
        +add_tools_to_model()
        +get_tools()
    }
    
    Agent "1" o-- "many" Function
    Agent "1" o-- "many" Toolkit
    Toolkit "1" o-- "many" Function
```

## 工具调用序列图

```mermaid
sequenceDiagram
    participant User as 用户
    participant Agent as Agent
    participant Model as 语言模型
    participant ToolRegistry as 工具注册表
    participant Tool as 工具函数
    
    User->>Agent: 发送需要工具的请求
    activate Agent
    
    Agent->>Model: 发送请求到模型
    activate Model
    
    Model-->>Agent: 返回工具调用请求
    deactivate Model
    
    Agent->>ToolRegistry: 查找工具
    activate ToolRegistry
    ToolRegistry-->>Agent: 返回工具定义
    deactivate ToolRegistry
    
    Agent->>Tool: 执行工具函数
    activate Tool
    Tool-->>Agent: 返回工具结果
    deactivate Tool
    
    Agent->>Model: 发送工具结果到模型
    activate Model
    Model-->>Agent: 返回最终响应
    deactivate Model
    
    Agent-->>User: 返回响应
    deactivate Agent
```

## 内置工具类型

```mermaid
graph TD
    A[Agent内置工具] --> B[聊天历史工具]
    A --> C[知识库搜索工具]
    A --> D[知识库更新工具]
    A --> E[工具调用历史工具]
    
    B --> B1[read_chat_history]
    C --> C1[search_knowledge_base]
    D --> D1[add_to_knowledge]
    E --> E1[get_tool_call_history]
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
```

## 工具定义示例

```python
from agno.agent import Agent
from agno.tools.function import Function

# 定义一个简单的计算器工具
def calculator(expression: str) -> float:
    """计算数学表达式的结果"""
    return eval(expression)

# 将函数转换为工具
calculator_tool = Function(
    name="calculator",
    description="计算数学表达式的结果",
    parameters={
        "type": "object",
        "properties": {
            "expression": {
                "type": "string",
                "description": "要计算的数学表达式，如 '2 + 2' 或 '3 * 4'"
            }
        },
        "required": ["expression"]
    },
    function=calculator
)

# 创建使用该工具的Agent
agent = Agent(
    name="计算助手",
    tools=[calculator_tool],
    show_tool_calls=True  # 在响应中显示工具调用
)

# 使用工具
response = agent.run("计算 (15 * 7) + (22 / 2) 的结果")
```

## 工具调用限制配置

```mermaid
flowchart LR
    A[Agent工具配置] --> B[设置tools列表]
    A --> C[设置tool_call_limit]
    A --> D[设置tool_choice]
    A --> E[设置show_tool_calls]
    
    B --> F[工具注册到模型]
    C --> F
    D --> F
    
    F --> G[模型使用工具]
    
    style F fill:#bbf,stroke:#333,stroke-width:2px
```

工具使用功能使Agent能够与外部世界交互，执行各种任务，从而大大增强了其实用性和功能范围。通过合理配置工具，可以创建出功能强大的专业Agent。 