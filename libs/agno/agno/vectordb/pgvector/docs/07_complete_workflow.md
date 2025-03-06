# PgVector 完整工作流程

本文档展示 PgVector 的完整工作流程，从初始化到搜索的全过程。

## 端到端工作流程

```mermaid
graph TD
    A[初始化PgVector] --> B[创建/连接数据库]
    B --> C[创建表和索引]
    C --> D[插入文档]
    D --> E[优化索引]
    E --> F[执行搜索]
    F --> G[处理搜索结果]
</mermaid>

## 详细工作流程

```mermaid
sequenceDiagram
    participant Client
    participant PgVector
    participant Embedder
    participant PostgreSQL
    
    Client->>PgVector: 创建PgVector实例
    PgVector->>PostgreSQL: 连接数据库
    
    alt 表不存在
        PgVector->>PostgreSQL: 创建表
    end
    
    Client->>PgVector: 插入文档
    
    loop 每个文档
        PgVector->>Embedder: 获取文档嵌入
        Embedder-->>PgVector: 返回嵌入向量
        PgVector->>PostgreSQL: 存储文档和嵌入
    end
    
    Client->>PgVector: 优化索引
    
    alt 使用HNSW索引
        PgVector->>PostgreSQL: 创建HNSW索引
    else 使用IVFFlat索引
        PgVector->>PostgreSQL: 创建IVFFlat索引
    end
    
    PgVector->>PostgreSQL: 创建GIN索引
    
    Client->>PgVector: 执行搜索查询
    
    alt 向量搜索
        PgVector->>Embedder: 获取查询嵌入
        Embedder-->>PgVector: 返回嵌入向量
        PgVector->>PostgreSQL: 执行向量相似度搜索
    else 关键词搜索
        PgVector->>PostgreSQL: 执行全文搜索
    else 混合搜索
        par 向量部分
            PgVector->>Embedder: 获取查询嵌入
            Embedder-->>PgVector: 返回嵌入向量
            PgVector->>PostgreSQL: 执行向量相似度搜索
        and 关键词部分
            PgVector->>PostgreSQL: 执行全文搜索
        end
        PgVector->>PgVector: 合并结果并计算混合得分
    end
    
    PostgreSQL-->>PgVector: 返回搜索结果
    PgVector->>PgVector: 转换为Document对象
    PgVector-->>Client: 返回文档列表
</mermaid>

## 代码示例工作流程

以下是使用 PgVector 的典型代码工作流程：

```mermaid
graph TD
    A[导入必要模块] --> B[创建嵌入器]
    B --> C[初始化PgVector]
    C --> D[创建表]
    D --> E[插入文档]
    E --> F[优化索引]
    F --> G[执行搜索]
    G --> H[处理结果]
</mermaid>

### 代码流程示例

```python
# 1. 导入必要模块
from agno.embedder.openai import OpenAIEmbedder
from agno.document import Document
from agno.vectordb.pgvector import PgVector
from agno.vectordb.pgvector.index import HNSW
from agno.vectordb.distance import Distance
from agno.vectordb.search import SearchType

# 2. 创建嵌入器
embedder = OpenAIEmbedder()

# 3. 初始化PgVector
vector_db = PgVector(
    table_name="documents",
    schema="ai",
    db_url="postgresql://user:password@localhost:5432/vectordb",
    embedder=embedder,
    search_type=SearchType.hybrid,
    vector_index=HNSW(ef_search=10, m=16),
    distance=Distance.cosine
)

# 4. 创建表
vector_db.create()

# 5. 插入文档
documents = [
    Document(id="doc1", name="文档1", content="这是第一个测试文档的内容"),
    Document(id="doc2", name="文档2", content="这是第二个测试文档的内容")
]
vector_db.insert(documents)

# 6. 优化索引
vector_db.optimize()

# 7. 执行搜索
results = vector_db.search("测试文档", limit=5)

# 8. 处理结果
for doc in results:
    print(f"ID: {doc.id}, 名称: {doc.name}, 内容: {doc.content}")
```

## 数据流图

```mermaid
flowchart TD
    A[客户端应用] -->|文档| B[PgVector]
    A -->|查询| B
    B -->|嵌入请求| C[嵌入器]
    C -->|嵌入向量| B
    B -->|SQL操作| D[PostgreSQL]
    D -->|查询结果| B
    B -->|文档结果| A
    
    subgraph PgVector内部
    E[文档处理] -->|文本| F[嵌入生成]
    F -->|向量| G[数据存储]
    H[查询处理] -->|查询文本| I[查询嵌入]
    I -->|查询向量| J[相似度搜索]
    K[索引管理] -->|优化| G
    end
</mermaid>

## 性能优化建议

```mermaid
graph TD
    A[性能优化] --> B[数据库级别]
    A --> C[应用级别]
    A --> D[索引级别]
    
    B --> B1[增加连接池大小]
    B --> B2[调整PostgreSQL配置]
    B --> B3[使用SSD存储]
    
    C --> C1[批量插入文档]
    C --> C2[异步处理嵌入]
    C --> C3[结果缓存]
    
    D --> D1[选择合适的索引类型]
    D --> D2[调整索引参数]
    D --> D3[定期重建索引]
</mermaid>

### 性能优化建议详解

1. **数据库级别**：
   - 增加连接池大小以处理并发请求
   - 调整 PostgreSQL 配置（shared_buffers, work_mem 等）
   - 使用 SSD 存储以提高 I/O 性能

2. **应用级别**：
   - 使用批量插入而非单条插入
   - 异步处理嵌入生成
   - 缓存常见查询结果

3. **索引级别**：
   - 根据数据规模选择合适的索引类型
   - 调整索引参数以平衡速度和准确性
   - 定期重建索引以保持性能 