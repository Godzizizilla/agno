# Reasoning 模型适配器

Reasoning模块支持多种不同的语言模型，通过专门的适配器来处理不同模型的特性和要求。本文档详细介绍Reasoning模块中的模型适配器，它们如何工作以及如何扩展支持新的模型。

## 模型适配器架构

```mermaid
flowchart TD
    A[推理请求] --> B[模型适配器选择]
    
    B --> C{模型类型}
    
    C -->|OpenAI| D[OpenAI适配器]
    C -->|Groq| E[Groq适配器]
    C -->|DeepSeek| F[DeepSeek适配器]
    C -->|默认| G[默认适配器]
    
    D --> H[OpenAI推理Agent]
    E --> I[Groq推理Agent]
    F --> J[DeepSeek推理Agent]
    G --> K[默认推理Agent]
    
    H --> L[执行推理]
    I --> L
    J --> L
    K --> L
    
    L --> M[标准化推理结果]
    M --> N[返回ReasoningSteps]
```

## 支持的模型适配器

Reasoning模块目前支持以下模型适配器：

1. **默认适配器** (default.py)
2. **OpenAI适配器** (openai.py)
3. **Groq适配器** (groq.py)
4. **DeepSeek适配器** (deepseek.py)

每个适配器都针对特定模型的特性进行了优化，确保最佳的推理性能和结果质量。

## 适配器组件详解

```mermaid
classDiagram
    class ModelAdapter {
        +get_reasoning_agent()
        +get_reasoning()
        +aget_reasoning()
    }
    
    class OpenAIAdapter {
        +get_openai_reasoning_agent()
        +get_openai_reasoning()
        +aget_openai_reasoning()
    }
    
    class GroqAdapter {
        +get_groq_reasoning_agent()
        +get_groq_reasoning()
        +aget_groq_reasoning()
    }
    
    class DeepSeekAdapter {
        +get_deepseek_reasoning_agent()
        +get_deepseek_reasoning()
        +aget_deepseek_reasoning()
    }
    
    class DefaultAdapter {
        +get_default_reasoning_agent()
    }
    
    ModelAdapter <|-- OpenAIAdapter
    ModelAdapter <|-- GroqAdapter
    ModelAdapter <|-- DeepSeekAdapter
    ModelAdapter <|-- DefaultAdapter
```

每个适配器通常包含以下核心功能：

1. **get_xxx_reasoning_agent()**: 创建一个针对特定模型优化的推理Agent
2. **get_xxx_reasoning()**: 同步执行推理过程并返回结果
3. **aget_xxx_reasoning()**: 异步执行推理过程并返回结果

## 默认适配器 (default.py)

默认适配器提供了一个通用的推理Agent实现，适用于大多数语言模型。它设置了详细的指令，指导模型如何执行结构化的推理过程。

```mermaid
flowchart LR
    A[默认适配器] --> B[创建推理Agent]
    B --> C[设置详细指令]
    C --> D[配置最小/最大步骤]
    D --> E[配置工具]
    E --> F[返回推理Agent]
```

核心功能：

```python
def get_default_reasoning_agent(
    reasoning_model: Model,
    min_steps: int,
    max_steps: int,
    tools: Optional[List[Union[Toolkit, Callable, Function]]] = None,
    structured_outputs: bool = False,
    monitoring: bool = False,
) -> Optional["Agent"]:
    # 创建并配置推理Agent
    # ...
```

## OpenAI适配器 (openai.py)

OpenAI适配器针对OpenAI模型（如GPT-3.5和GPT-4）进行了优化，利用这些模型的特性来提供高质量的推理结果。

```mermaid
sequenceDiagram
    participant Agent as 主Agent
    participant OpenAIAdapter as OpenAI适配器
    participant OpenAIModel as OpenAI模型
    
    Agent->>OpenAIAdapter: 请求推理
    activate OpenAIAdapter
    
    OpenAIAdapter->>OpenAIAdapter: 格式化消息
    OpenAIAdapter->>OpenAIModel: 发送请求
    activate OpenAIModel
    
    OpenAIModel->>OpenAIModel: 执行推理
    OpenAIModel-->>OpenAIAdapter: 返回响应
    deactivate OpenAIModel
    
    OpenAIAdapter->>OpenAIAdapter: 提取思考内容
    OpenAIAdapter-->>Agent: 返回推理结果
    deactivate OpenAIAdapter
```

核心功能：

```python
def get_openai_reasoning(reasoning_agent: "Agent", messages: List[Message]) -> Optional[Message]:
    # 执行OpenAI推理并处理结果
    # ...
```

## Groq适配器 (groq.py)

Groq适配器针对Groq模型进行了优化，利用Groq的高性能特性来提供快速的推理结果。

```mermaid
flowchart TD
    A[Groq适配器] --> B[创建Groq推理Agent]
    B --> C[配置Groq特定参数]
    C --> D[执行推理请求]
    D --> E[处理Groq响应]
    E --> F[提取推理内容]
    F --> G[返回标准化结果]
```

核心功能：

```python
def get_groq_reasoning_agent(reasoning_model: Model, **kwargs) -> "Agent":
    # 创建并配置Groq推理Agent
    # ...

def get_groq_reasoning(reasoning_agent: "Agent", messages: List[Message]) -> Optional[Message]:
    # 执行Groq推理并处理结果
    # ...
```

## DeepSeek适配器 (deepseek.py)

DeepSeek适配器针对DeepSeek模型进行了优化，利用这些模型的特性来提供高质量的推理结果。

```mermaid
sequenceDiagram
    participant Agent as 主Agent
    participant DeepSeekAdapter as DeepSeek适配器
    participant DeepSeekModel as DeepSeek模型
    
    Agent->>DeepSeekAdapter: 请求推理
    activate DeepSeekAdapter
    
    DeepSeekAdapter->>DeepSeekAdapter: 格式化消息
    DeepSeekAdapter->>DeepSeekModel: 发送请求
    activate DeepSeekModel
    
    DeepSeekModel->>DeepSeekModel: 执行推理
    DeepSeekModel-->>DeepSeekAdapter: 返回响应
    deactivate DeepSeekModel
    
    DeepSeekAdapter->>DeepSeekAdapter: 提取思考内容
    DeepSeekAdapter-->>Agent: 返回推理结果
    deactivate DeepSeekAdapter
```

核心功能：

```python
def get_deepseek_reasoning(reasoning_agent: "Agent", messages: List[Message]) -> Optional[Message]:
    # 执行DeepSeek推理并处理结果
    # ...
```

## 适配器选择逻辑

```mermaid
flowchart TD
    A[Agent配置] --> B[检查reasoning_model类型]
    
    B --> C{模型类型?}
    
    C -->|OpenAIChat| D[使用OpenAI适配器]
    C -->|GroqChat| E[使用Groq适配器]
    C -->|DeepSeekChat| F[使用DeepSeek适配器]
    C -->|其他| G[使用默认适配器]
    
    D --> H[创建并返回适配的推理Agent]
    E --> H
    F --> H
    G --> H
```

当Agent配置启用推理功能时，系统会根据提供的reasoning_model类型自动选择合适的适配器。

## 模型特定优化

每个适配器都包含针对特定模型的优化：

1. **OpenAI适配器**:
   - 处理思考标签 (`<think>...</think>`)
   - 适应OpenAI的消息格式
   - 支持异步操作

2. **Groq适配器**:
   - 针对Groq的高速处理进行优化
   - 适应Groq的API特性
   - 支持异步操作

3. **DeepSeek适配器**:
   - 针对DeepSeek模型的特性进行优化
   - 适应DeepSeek的消息格式
   - 支持异步操作

## 扩展支持新模型

要为Reasoning模块添加新的模型支持，需要创建一个新的适配器文件，实现以下核心功能：

```python
# 示例：为新模型创建适配器
def get_newmodel_reasoning_agent(reasoning_model: Model, **kwargs) -> "Agent":
    # 创建并配置新模型的推理Agent
    # ...

def get_newmodel_reasoning(reasoning_agent: "Agent", messages: List[Message]) -> Optional[Message]:
    # 执行新模型的推理并处理结果
    # ...

async def aget_newmodel_reasoning(reasoning_agent: "Agent", messages: List[Message]) -> Optional[Message]:
    # 异步执行新模型的推理并处理结果
    # ...
```

然后，在Agent系统中添加对新适配器的支持，使其能够根据模型类型自动选择正确的适配器。

## 适配器的优势

1. **模型特定优化**: 每个适配器都针对特定模型的特性进行了优化
2. **统一接口**: 所有适配器提供统一的接口，简化集成
3. **可扩展性**: 易于添加新模型的支持
4. **同步和异步支持**: 支持同步和异步操作，提高灵活性
5. **标准化输出**: 所有适配器返回标准化的推理结果，便于后续处理

通过这种适配器架构，Reasoning模块能够支持多种不同的语言模型，同时保持一致的接口和行为，使Agent能够灵活地选择最适合特定任务的模型。 