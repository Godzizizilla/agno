# 文档嵌入过程详解

文档嵌入是将文本内容转换为向量表示的过程，这些向量可以用于语义搜索、相似度计算和其他自然语言处理任务。

## 嵌入流程

```mermaid
flowchart TD
    A[Document对象] --> B[调用embed方法]
    B --> C{是否提供embedder?}
    C -- 是 --> D[使用提供的embedder]
    C -- 否 --> E[使用Document自身的embedder]
    E --> F{embedder是否为空?}
    F -- 是 --> G[抛出ValueError]
    F -- 否 --> H[继续处理]
    D --> H
    H --> I[调用get_embedding_and_usage方法]
    I --> J[获取embedding和usage]
    J --> K[更新Document的embedding和usage属性]
    K --> L[完成嵌入]
```

## 嵌入过程时序图

```mermaid
sequenceDiagram
    participant App
    participant Doc as Document
    participant Emb as Embedder
    participant API as 嵌入API
    
    App->>Doc: embed(embedder)
    
    alt 提供了embedder
        Doc->>Doc: 使用提供的embedder
    else 未提供embedder
        Doc->>Doc: 使用自身的embedder
        alt embedder为空
            Doc-->>App: 抛出ValueError
        end
    end
    
    Doc->>Emb: get_embedding_and_usage(content)
    Emb->>API: 调用嵌入API
    API-->>Emb: 返回嵌入向量和使用信息
    Emb-->>Doc: 返回embedding和usage
    Doc->>Doc: 更新embedding和usage属性
    Doc-->>App: 完成嵌入
```

## 支持的嵌入模型

Agno框架支持多种嵌入模型，包括但不限于：

1. **OpenAI嵌入模型**：如text-embedding-ada-002、text-embedding-3-small等
2. **Hugging Face模型**：如sentence-transformers系列模型
3. **自定义嵌入模型**：用户可以实现自己的嵌入器

## 嵌入向量的应用

```mermaid
graph TD
    A[文档嵌入向量] --> B[语义搜索]
    A --> C[文档聚类]
    A --> D[相似度计算]
    A --> E[信息检索]
    A --> F[问答系统]
    
    B --> G[找到相关文档]
    C --> H[发现主题组]
    D --> I[比较文档相似性]
    E --> J[检索相关信息]
    F --> K[生成准确回答]
```

## 嵌入示例

```python
from agno.document import Document
from agno.embedder import OpenAIEmbedder

# 创建Document实例
doc = Document(
    content="这是一个示例文档，用于演示嵌入过程。",
    name="embedding_example"
)

# 创建嵌入器
embedder = OpenAIEmbedder(model="text-embedding-3-small")

# 嵌入文档
doc.embed(embedder)

# 查看嵌入结果
print(f"嵌入维度: {len(doc.embedding)}")
print(f"使用信息: {doc.usage}")
``` 