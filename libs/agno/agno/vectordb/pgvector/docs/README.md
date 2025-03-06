# PgVector 文档

欢迎使用 PgVector 文档！本文档集详细介绍了 PgVector 的架构、功能和使用方法。

## 什么是 PgVector？

PgVector 是一个基于 PostgreSQL 和 pgvector 扩展的向量数据库接口，用于高效存储和检索向量嵌入。它提供了多种搜索方式和索引优化选项，适用于各种向量相似度搜索场景。

```mermaid
graph LR
    A[应用程序] --> B[PgVector]
    B --> C[PostgreSQL + pgvector]
    B --> D[嵌入模型]
```

## 文档目录

本文档集包含以下内容：

1. [**概述**](01_overview.md) - PgVector 的基本架构和主要组件
2. [**初始化流程**](02_initialization.md) - PgVector 的初始化流程和配置选项
3. [**文档操作**](03_document_operations.md) - 文档的插入、更新和删除操作
4. [**搜索操作**](04_search_operations.md) - 向量搜索、关键词搜索和混合搜索
5. [**索引优化**](05_index_optimization.md) - HNSW 和 IVFFlat 索引的创建和管理
6. [**数据库管理**](06_database_management.md) - 表创建、检查和删除
7. [**完整工作流程**](07_complete_workflow.md) - 从初始化到搜索的全过程

## 核心功能概览

```mermaid
mindmap
  root((PgVector))
    数据库管理
      表创建
      表检查
      表删除
    文档操作
      插入
      更新
      删除
      批量处理
    搜索功能
      向量搜索
      关键词搜索
      混合搜索
      过滤器应用
    索引优化
      HNSW索引
      IVFFlat索引
      GIN索引
    配置选项
      距离度量
      搜索类型
      索引参数
```

## 快速开始

以下是使用 PgVector 的基本步骤：

1. 安装必要的依赖：
   - PostgreSQL 数据库
   - pgvector 扩展
   - SQLAlchemy
   - 嵌入模型（如 OpenAI 嵌入模型）

2. 初始化 PgVector：
   ```python
   from agno.embedder.openai import OpenAIEmbedder
   from agno.vectordb.pgvector import PgVector
   
   embedder = OpenAIEmbedder()
   vector_db = PgVector(
       table_name="documents",
       schema="ai",
       db_url="postgresql://user:password@localhost:5432/vectordb",
       embedder=embedder
   )
   ```

3. 创建表并插入文档：
   ```python
   from agno.document import Document
   
   vector_db.create()
   
   documents = [
       Document(id="doc1", name="文档1", content="这是第一个测试文档的内容"),
       Document(id="doc2", name="文档2", content="这是第二个测试文档的内容")
   ]
   vector_db.insert(documents)
   ```

4. 优化索引并执行搜索：
   ```python
   vector_db.optimize()
   results = vector_db.search("测试文档", limit=5)
   
   for doc in results:
       print(f"ID: {doc.id}, 名称: {doc.name}, 内容: {doc.content}")
   ```

## 技术架构

```mermaid
graph TD
    A[PgVector] --> B[数据库连接层]
    A --> C[文档处理层]
    A --> D[搜索引擎层]
    A --> E[索引管理层]
    
    B --> F[SQLAlchemy]
    F --> G[PostgreSQL]
    G --> H[pgvector扩展]
    
    C --> I[文档转换]
    C --> J[嵌入生成]
    
    D --> K[向量搜索]
    D --> L[全文搜索]
    D --> M[混合搜索]
    
    E --> N[HNSW索引]
    E --> O[IVFFlat索引]
    E --> P[GIN索引]
```

## 贡献与反馈

如果您有任何问题、建议或贡献，请联系项目维护者。 