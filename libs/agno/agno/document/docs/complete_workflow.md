# Document模块完整工作流程

本文档描述了使用Agno框架的Document模块处理文档的完整工作流程，从文档读取到分块、嵌入和应用。

## 整体流程

```mermaid
flowchart TD
    A[文档来源] --> B[Reader读取文档]
    B --> C[创建Document对象]
    C --> D{是否需要分块?}
    D -- 是 --> E[选择分块策略]
    E --> F[执行文档分块]
    D -- 否 --> G[保持原始文档]
    F --> H[文档嵌入]
    G --> H
    H --> I[文档应用]
    I --> J1[语义搜索]
    I --> J2[问答系统]
    I --> J3[文档分析]
    I --> J4[知识库构建]
```

## 详细工作流程

```mermaid
sequenceDiagram
    participant Source as 文档来源
    participant Reader
    participant Document
    participant Chunking as 分块策略
    participant Embedder
    participant Application as 应用场景
    
    Source->>Reader: 提供文档
    Reader->>Reader: 读取文档内容
    Reader->>Document: 创建Document对象
    
    alt 需要分块
        Document->>Chunking: 传递文档
        Chunking->>Chunking: 执行分块逻辑
        Chunking->>Document: 返回分块后的Document列表
    end
    
    Document->>Embedder: 请求嵌入
    Embedder->>Embedder: 计算嵌入向量
    Embedder->>Document: 返回嵌入向量
    Document->>Application: 提供处理后的文档
```

## 使用示例：从PDF到问答系统

```mermaid
graph TD
    A[PDF文件] --> B[PDFReader]
    B --> C[创建Document对象]
    C --> D[RecursiveChunking]
    D --> E[分块后的Document列表]
    E --> F[OpenAIEmbedder]
    F --> G[嵌入后的Document列表]
    G --> H[存储到向量数据库]
    H --> I[问答系统]
    I --> J[用户查询]
    J --> K[检索相关文档]
    K --> L[生成回答]
```

## 代码示例

```python
from agno.document import Document
from agno.document.reader import PDFReader
from agno.document.chunking import RecursiveChunking
from agno.embedder import OpenAIEmbedder
from agno.vectordb import ChromaDB

# 1. 读取PDF文档
pdf_reader = PDFReader(chunk=False)  # 先不分块
documents = pdf_reader.read("example.pdf")

# 2. 分块处理
chunking_strategy = RecursiveChunking(chunk_size=1000)
chunked_documents = []
for doc in documents:
    chunked_documents.extend(chunking_strategy.chunk(doc))

# 3. 嵌入文档
embedder = OpenAIEmbedder()
for doc in chunked_documents:
    doc.embed(embedder)

# 4. 存储到向量数据库
vector_db = ChromaDB(collection_name="example_docs")
vector_db.add_documents(chunked_documents)

# 5. 检索相关文档
query = "什么是文档处理系统?"
relevant_docs = vector_db.similarity_search(query, k=3)

# 6. 使用检索到的文档生成回答
# ...
```

## 各阶段关键点

1. **文档读取阶段**
   - 选择合适的Reader
   - 处理不同格式的文档
   - 提取文档内容和元数据

2. **文档分块阶段**
   - 选择合适的分块策略
   - 设置合适的分块大小和重叠
   - 保持文档的语义完整性

3. **文档嵌入阶段**
   - 选择合适的嵌入模型
   - 处理嵌入向量
   - 管理嵌入使用信息

4. **文档应用阶段**
   - 存储和检索文档
   - 计算文档相似度
   - 构建知识库和问答系统 