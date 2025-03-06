# PgVector 概述

PgVector 是一个基于 PostgreSQL 和 pgvector 扩展的向量数据库接口，用于高效存储和检索向量嵌入。它提供了多种搜索方式和索引优化选项，适用于各种向量相似度搜索场景。

## 主要组件

```mermaid
graph TD
    A[PgVector 类] --> B[数据库连接管理]
    A --> C[文档管理]
    A --> D[向量搜索]
    A --> E[索引优化]
    
    B --> B1[创建/连接数据库]
    B --> B2[表管理]
    
    C --> C1[插入文档]
    C --> C2[更新文档]
    C --> C3[删除文档]
    
    D --> D1[向量搜索]
    D --> D2[关键词搜索]
    D --> D3[混合搜索]
    
    E --> E1[HNSW索引]
    E --> E2[IVFFlat索引]
    E --> E3[GIN索引]
```

## 核心功能

PgVector 提供以下核心功能：

1. **向量存储**：将文档内容转换为向量嵌入并存储在 PostgreSQL 数据库中
2. **多种搜索方式**：支持向量搜索、关键词搜索和混合搜索
3. **索引优化**：支持 HNSW 和 IVFFlat 两种向量索引类型，提高搜索效率
4. **文档管理**：提供文档的增删改查功能

## 技术依赖

```mermaid
graph LR
    PgVector --> SQLAlchemy[SQLAlchemy]
    PgVector --> PgVectorExt[pgvector扩展]
    PgVector --> PostgreSQL[PostgreSQL数据库]
    PgVector --> Embedder[嵌入模型]
```

PgVector 依赖于以下技术组件：

- **PostgreSQL**：底层关系型数据库
- **pgvector 扩展**：PostgreSQL 的向量扩展，提供向量数据类型和向量操作
- **SQLAlchemy**：Python SQL 工具包和 ORM
- **嵌入模型**：用于将文本转换为向量嵌入的模型（如 OpenAI 嵌入模型） 