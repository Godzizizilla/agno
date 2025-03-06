# Document类详解

Document类是整个文档处理系统的核心数据结构，它封装了文档的内容、元数据和嵌入向量等信息。

## 类结构

```mermaid
classDiagram
    class Document {
        +str content
        +str id
        +str name
        +Dict meta_data
        +Embedder embedder
        +List[float] embedding
        +Dict usage
        +float reranking_score
        +embed(embedder)
        +to_dict()
        +from_dict(document)
        +from_json(document)
    }
```

## 属性说明

- **content**: 文档的文本内容
- **id**: 文档的唯一标识符
- **name**: 文档的名称
- **meta_data**: 与文档相关的元数据，如页码、来源等
- **embedder**: 用于生成文档嵌入的嵌入器
- **embedding**: 文档内容的向量表示
- **usage**: 与嵌入相关的使用信息
- **reranking_score**: 重排序得分，用于文档检索排序

## 方法说明

```mermaid
sequenceDiagram
    participant App
    participant Doc as Document
    participant Emb as Embedder
    
    App->>Doc: 创建Document实例
    App->>Doc: embed(embedder)
    Doc->>Emb: get_embedding_and_usage(content)
    Emb-->>Doc: 返回embedding和usage
    App->>Doc: to_dict()
    Doc-->>App: 返回字典表示
    App->>Doc: from_dict(document)
    Doc-->>App: 返回Document实例
    App->>Doc: from_json(document)
    Doc-->>App: 返回Document实例
```

## 使用示例

```python
# 创建Document实例
doc = Document(
    content="这是一个示例文档",
    name="example",
    meta_data={"source": "user_input"}
)

# 使用嵌入器生成嵌入
from agno.embedder import OpenAIEmbedder
embedder = OpenAIEmbedder()
doc.embed(embedder)

# 转换为字典
doc_dict = doc.to_dict()

# 从字典创建Document
new_doc = Document.from_dict(doc_dict)
```