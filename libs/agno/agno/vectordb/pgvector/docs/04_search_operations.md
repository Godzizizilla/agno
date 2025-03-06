# PgVector 搜索操作

本文档详细说明 PgVector 类的搜索操作流程，包括向量搜索、关键词搜索和混合搜索。

## 搜索类型概述

PgVector 支持三种搜索类型：

1. **向量搜索**：基于向量相似度的搜索
2. **关键词搜索**：基于全文检索的搜索
3. **混合搜索**：结合向量相似度和全文检索的搜索

```mermaid
graph TD
    A[搜索请求] --> B{搜索类型?}
    B -->|向量搜索| C[vector_search]
    B -->|关键词搜索| D[keyword_search]
    B -->|混合搜索| E[hybrid_search]
    
    C --> F[返回结果]
    D --> F
    E --> F
```

## 向量搜索流程

```mermaid
sequenceDiagram
    participant Client
    participant PgVector
    participant Embedder
    participant Database
    
    Client->>PgVector: vector_search(query, limit, filters)
    PgVector->>Embedder: 获取查询嵌入
    Embedder-->>PgVector: 返回嵌入向量
    
    PgVector->>PgVector: 构建查询语句
    Note right of PgVector: 选择列<br>应用过滤器<br>设置距离度量<br>设置结果限制
    
    alt 使用向量索引
        PgVector->>Database: 设置索引参数
        Note right of Database: IVFFlat: 设置probes<br>HNSW: 设置ef_search
    end
    
    PgVector->>Database: 执行查询
    Database-->>PgVector: 返回结果
    
    PgVector->>PgVector: 转换为Document对象
    PgVector-->>Client: 返回文档列表
```

### 向量搜索过程详解

1. **获取查询嵌入**：将查询文本转换为向量嵌入
2. **构建查询语句**：
   - 选择需要的列
   - 应用过滤器（如果提供）
   - 根据距离度量（L2、余弦、最大内积）排序
   - 限制结果数量
3. **设置索引参数**：根据索引类型设置搜索参数
4. **执行查询**：执行 SQL 查询并获取结果
5. **处理结果**：将数据库结果转换为 Document 对象列表

## 关键词搜索流程

```mermaid
flowchart TD
    A[关键词搜索] --> B[清理查询文本]
    B --> C{启用前缀匹配?}
    C -->|是| D[添加前缀匹配语法]
    C -->|否| E[保持原查询]
    D --> F[构建全文搜索查询]
    E --> F
    F --> G[应用过滤器]
    G --> H[执行查询]
    H --> I[转换结果为Document]
    I --> J[返回结果]
```

### 关键词搜索过程详解

1. **清理查询文本**：处理特殊字符和格式
2. **前缀匹配处理**：如果启用前缀匹配，添加相应语法
3. **构建全文搜索查询**：使用 PostgreSQL 的 `to_tsquery` 和 `ts_rank`
4. **应用过滤器**：如果提供了过滤器，则应用到查询中
5. **执行查询**：执行 SQL 查询并获取结果
6. **处理结果**：将数据库结果转换为 Document 对象列表

## 混合搜索流程

```mermaid
sequenceDiagram
    participant Client
    participant PgVector
    participant Embedder
    participant Database
    
    Client->>PgVector: hybrid_search(query, limit, filters)
    
    par 向量搜索
        PgVector->>Embedder: 获取查询嵌入
        Embedder-->>PgVector: 返回嵌入向量
        PgVector->>Database: 执行向量相似度查询
        Database-->>PgVector: 返回向量搜索结果
    and 关键词搜索
        PgVector->>PgVector: 处理查询文本
        PgVector->>Database: 执行全文搜索查询
        Database-->>PgVector: 返回关键词搜索结果
    end
    
    PgVector->>PgVector: 合并结果并计算混合得分
    Note right of PgVector: 向量得分 * 权重 +<br>关键词得分 * (1-权重)
    
    PgVector->>PgVector: 按混合得分排序
    PgVector->>PgVector: 限制结果数量
    
    PgVector-->>Client: 返回文档列表
```

### 混合搜索过程详解

1. **并行执行搜索**：
   - 执行向量相似度搜索
   - 执行全文关键词搜索
2. **合并结果**：将两种搜索结果合并
3. **计算混合得分**：
   - 向量得分 × 向量权重
   - 关键词得分 × (1 - 向量权重)
4. **排序和限制**：按混合得分排序并限制结果数量
5. **返回结果**：返回排序后的 Document 对象列表

## 距离度量选项

```mermaid
graph LR
    A[距离度量] --> B[L2距离]
    A --> C[余弦距离]
    A --> D[最大内积]
    
    B --> E[欧几里得距离]
    C --> F[1 - 余弦相似度]
    D --> G[负点积]
```

PgVector 支持三种向量距离度量：

1. **L2 距离**：欧几里得距离，适用于欧几里得空间中的向量
2. **余弦距离**：1 减去余弦相似度，适用于方向相似性
3. **最大内积**：负点积，适用于大小和方向都重要的情况 