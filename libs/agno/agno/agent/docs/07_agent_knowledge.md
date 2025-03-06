# Agent 知识库系统

Agent的知识库系统允许它访问和利用外部知识，通过检索增强生成(RAG)技术提高回答的准确性和相关性。

## 知识库系统架构

```mermaid
graph TD
    A[Agent知识库系统] --> B[文档存储]
    A --> C[向量数据库]
    A --> D[检索系统]
    A --> E[知识更新]
    
    B --> B1[文本文档]
    B --> B2[结构化数据]
    
    C --> C1[嵌入向量]
    C --> C2[相似度搜索]
    
    D --> D1[语义检索]
    D --> D2[混合检索]
    
    E --> E1[知识添加]
    E --> E2[知识更新]
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
```

## RAG检索流程

```mermaid
flowchart TD
    A[用户查询] --> B[查询处理]
    B --> C[生成查询向量]
    C --> D[向量数据库搜索]
    D --> E[获取相关文档]
    E --> F[文档排序]
    F --> G[添加到用户消息]
    G --> H[模型生成回答]
```

## 知识检索序列图

```mermaid
sequenceDiagram
    participant User as 用户
    participant Agent as Agent
    participant Knowledge as 知识库
    participant VectorDB as 向量数据库
    participant Model as 语言模型
    
    User->>Agent: 发送查询
    activate Agent
    
    opt 启用知识检索
        Agent->>Knowledge: 检索相关知识
        activate Knowledge
        
        Knowledge->>VectorDB: 向量搜索
        activate VectorDB
        VectorDB-->>Knowledge: 返回相似文档
        deactivate VectorDB
        
        Knowledge-->>Agent: 返回相关文档
        deactivate Knowledge
        
        Agent->>Agent: 将文档添加到用户消息
    end
    
    Agent->>Model: 发送增强后的消息
    activate Model
    Model-->>Agent: 返回基于知识的回答
    deactivate Model
    
    Agent-->>User: 返回响应
    deactivate Agent
```

## 知识库数据结构

```mermaid
classDiagram
    class AgentKnowledge {
        +VectorStore vector_store
        +Embeddings embeddings
        +add_texts(texts)
        +add_documents(documents)
        +similarity_search(query)
        +max_tokens_limit
    }
    
    class Document {
        +String content
        +Dict metadata
        +String id
    }
    
    class VectorStore {
        +add_documents(documents)
        +similarity_search(query, k)
        +delete(ids)
    }
    
    class Embeddings {
        +embed_query(text)
        +embed_documents(documents)
    }
    
    AgentKnowledge "1" -- "1" VectorStore
    AgentKnowledge "1" -- "1" Embeddings
    VectorStore "1" o-- "many" Document
```

## 知识库使用模式

```mermaid
flowchart LR
    A[知识库使用模式] --> B[自动RAG]
    A --> C[工具化RAG]
    A --> D[混合模式]
    
    B --> B1[add_references=True]
    C --> C1[search_knowledge=True]
    D --> D1[两者结合使用]
    
    B1 --> E[自动添加到用户消息]
    C1 --> F[模型决定何时检索]
    D1 --> G[灵活使用知识]
```

## 知识库代码示例

```python
from agno.agent import Agent
from agno.knowledge.agent import AgentKnowledge
from agno.knowledge.vector_stores import ChromaVectorStore
from agno.knowledge.embeddings import OpenAIEmbeddings

# 创建知识库
knowledge = AgentKnowledge(
    vector_store=ChromaVectorStore(collection_name="my_docs"),
    embeddings=OpenAIEmbeddings()
)

# 添加文档到知识库
knowledge.add_texts([
    "人工智能是计算机科学的一个分支，致力于创建能够模拟人类智能的系统。",
    "机器学习是人工智能的一个子领域，专注于使计算机系统能够从数据中学习和改进。",
    "深度学习是机器学习的一种特定方法，使用神经网络进行学习。"
])

# 创建使用知识库的Agent
agent = Agent(
    name="知识助手",
    knowledge=knowledge,
    # 自动RAG模式
    add_references=True,
    # 也可以作为工具使用
    search_knowledge=True
)

# 使用知识回答问题
response = agent.run("请解释什么是深度学习?")
```

## 知识库更新流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant Agent as Agent
    participant Knowledge as 知识库
    participant VectorDB as 向量数据库
    
    User->>Agent: 请添加新知识
    activate Agent
    
    Agent->>Knowledge: 添加新文档
    activate Knowledge
    
    Knowledge->>Knowledge: 生成文档嵌入
    
    Knowledge->>VectorDB: 存储文档和嵌入
    activate VectorDB
    VectorDB-->>Knowledge: 确认存储
    deactivate VectorDB
    
    Knowledge-->>Agent: 确认添加
    deactivate Knowledge
    
    Agent-->>User: 知识已添加
    deactivate Agent
```

知识库系统使Agent能够访问和利用大量外部信息，大大增强了其回答问题的能力，特别是在处理专业领域或需要最新信息的查询时。 