# Agent 推理功能

Agent的推理功能允许它通过多步骤思考来解决复杂问题，这是一种类似于人类思考的方式，可以提高Agent的问题解决能力。

## 推理流程概述

```mermaid
flowchart TD
    A[接收用户问题] --> B[启动推理流程]
    B --> C[执行第一步推理]
    
    C --> D[分析当前状态]
    D --> E[确定下一步行动]
    E --> F{是否需要工具?}
    
    F -->|是| G[调用工具]
    G --> H[分析工具结果]
    H --> D
    
    F -->|否| I{是否达到结论?}
    I -->|否| D
    I -->|是| J[生成最终答案]
    J --> K[返回完整推理过程和答案]
```

## 推理步骤详解

```mermaid
sequenceDiagram
    participant User as 用户
    participant Agent as 主Agent
    participant Reasoning as 推理Agent
    participant Tools as 工具系统
    
    User->>Agent: 发送复杂问题
    activate Agent
    
    Agent->>Reasoning: 启动推理流程
    activate Reasoning
    
    loop 推理步骤
        Reasoning->>Reasoning: 思考当前状态
        Reasoning->>Reasoning: 确定下一步行动
        
        alt 需要使用工具
            Reasoning->>Tools: 调用工具
            activate Tools
            Tools-->>Reasoning: 返回工具结果
            deactivate Tools
        else 继续思考
            Reasoning->>Reasoning: 深入分析问题
        end
        
        Reasoning->>Reasoning: 评估是否达到结论
    end
    
    Reasoning->>Reasoning: 整合所有步骤
    Reasoning-->>Agent: 返回推理过程和结论
    deactivate Reasoning
    
    Agent-->>User: 返回最终答案(可选包含推理过程)
    deactivate Agent
```

## 推理步骤数据结构

```mermaid
classDiagram
    class ReasoningStep {
        +String thinking
        +NextAction next_action
        +Dict tool_calls
        +Dict tool_results
        +Bool is_last
    }
    
    class NextAction {
        +String type
        +Dict details
    }
    
    class ReasoningSteps {
        +List~ReasoningStep~ steps
        +add_step(step)
        +get_last_step()
        +get_thinking_process()
    }
    
    ReasoningSteps "1" o-- "many" ReasoningStep
    ReasoningStep "1" *-- "1" NextAction
```

## 推理模式配置

```mermaid
flowchart LR
    A[Agent配置] --> B{启用推理?}
    B -->|是| C[配置推理参数]
    B -->|否| D[标准模式运行]
    
    C --> E[设置reasoning=True]
    C --> F[配置reasoning_model]
    C --> G[设置reasoning_min_steps]
    C --> H[设置reasoning_max_steps]
    
    E --> I[启动推理模式]
    F --> I
    G --> I
    H --> I
```

## 推理示例代码

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

# 创建一个启用推理功能的Agent
agent = Agent(
    model=OpenAIChat(model="gpt-4"),
    name="推理助手",
    reasoning=True,  # 启用推理功能
    reasoning_min_steps=1,  # 最少推理步骤
    reasoning_max_steps=5,  # 最多推理步骤
    # 可以指定不同的推理模型
    reasoning_model=OpenAIChat(model="gpt-4"),
    # 显示推理过程
    show_reasoning=True
)

# 使用推理功能解决问题
response = agent.run("解决这个复杂的数学问题: 如果一个圆的周长是10π，那么它的面积是多少?")
```

推理功能使Agent能够像人类一样逐步思考问题，特别适合解决需要多步骤分析的复杂任务，如数学问题、逻辑推理、规划等。 