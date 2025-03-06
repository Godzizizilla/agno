# Chunking系统详解

Chunking系统负责将长文档分割成更小的片段，以便于处理和分析。Agno框架提供了多种分块策略，适用于不同的场景和需求。

## Chunking策略层次结构

```mermaid
classDiagram
    class ChunkingStrategy {
        <<abstract>>
        +chunk(document) List[Document]
        +clean_text(text) str
    }
    
    ChunkingStrategy <|-- FixedSizeChunking
    ChunkingStrategy <|-- RecursiveChunking
    ChunkingStrategy <|-- SemanticChunking
    ChunkingStrategy <|-- AgenticChunking
    
    class FixedSizeChunking {
        +int chunk_size
        +int overlap
        +chunk(document) List[Document]
    }
    
    class RecursiveChunking {
        +int chunk_size
        +List[str] separators
        +chunk(document) List[Document]
    }
    
    class SemanticChunking {
        +int chunk_size
        +float similarity_threshold
        +chunk(document) List[Document]
    }
    
    class AgenticChunking {
        +LLM llm
        +chunk(document) List[Document]
    }
```

## 分块流程

```mermaid
flowchart TD
    A[开始] --> B[选择分块策略]
    B --> C[调用chunk方法]
    C --> D[清理文本]
    D --> E[根据策略分块]
    E --> F[创建新的Document对象]
    F --> G[返回Document列表]
    G --> H[结束]
```

## 分块策略比较

```mermaid
graph LR
    A[文档] --> B[FixedSizeChunking]
    A --> C[RecursiveChunking]
    A --> D[SemanticChunking]
    A --> E[AgenticChunking]
    
    B --> F[固定大小的块]
    C --> G[基于分隔符的层次块]
    D --> H[语义相关的块]
    E --> I[基于LLM的智能块]
    
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style C fill:#bbf,stroke:#333,stroke-width:2px
    style D fill:#bfb,stroke:#333,stroke-width:2px
    style E fill:#fbf,stroke:#333,stroke-width:2px
```

## 分块策略说明

### FixedSizeChunking

固定大小分块策略将文档按照指定的大小（字符数）分割成多个块，可以设置重叠区域以保持上下文连贯性。

```mermaid
sequenceDiagram
    participant Doc as Document
    participant Fixed as FixedSizeChunking
    
    Doc->>Fixed: chunk(document)
    Fixed->>Fixed: 清理文本
    Fixed->>Fixed: 按固定大小分割
    Fixed->>Fixed: 调整分割点（避免切分单词）
    Fixed->>Fixed: 创建新Document对象
    Fixed-->>Doc: 返回分块后的Document列表
```

### RecursiveChunking

递归分块策略根据一系列分隔符（如段落、句子、单词等）递归地将文档分割成层次化的块。

### SemanticChunking

语义分块策略根据文本的语义相似性将文档分割成语义连贯的块，需要使用嵌入模型计算文本片段之间的相似度。

### AgenticChunking

智能分块策略使用大型语言模型（LLM）来智能地将文档分割成有意义的块，可以理解文档的结构和内容。 