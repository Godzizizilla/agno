# PgVector 索引优化

本文档详细说明 PgVector 类的索引优化流程，包括 HNSW 和 IVFFlat 索引的创建和管理。

## 索引类型概述

PgVector 支持两种主要的向量索引类型：

1. **HNSW（分层可导航小世界）**：基于图的索引结构，适合高维向量搜索
2. **IVFFlat（倒排文件平面）**：基于聚类的索引结构，适合大规模数据集

此外，还支持 GIN 索引用于全文搜索优化。

```mermaid
graph TD
    A[索引优化] --> B[向量索引]
    A --> C[全文搜索索引]
    
    B --> D[HNSW索引]
    B --> E[IVFFlat索引]
    
    C --> F[GIN索引]
</mermaid>

## 索引优化流程

```mermaid
sequenceDiagram
    participant User
    participant PgVector
    participant Database
    
    User->>PgVector: optimize(force_recreate)
    
    PgVector->>PgVector: _create_vector_index(force_recreate)
    
    alt 向量索引存在且force_recreate=true
        PgVector->>Database: 删除现有索引
        Database-->>PgVector: 索引已删除
    end
    
    alt 向量索引不存在或已删除
        alt 索引类型=HNSW
            PgVector->>Database: 创建HNSW索引
        else 索引类型=IVFFlat
            PgVector->>Database: 创建IVFFlat索引
        end
        Database-->>PgVector: 索引已创建
    end
    
    PgVector->>PgVector: _create_gin_index(force_recreate)
    
    alt GIN索引存在且force_recreate=true
        PgVector->>Database: 删除现有GIN索引
        Database-->>PgVector: GIN索引已删除
    end
    
    alt GIN索引不存在或已删除
        PgVector->>Database: 创建GIN索引
        Database-->>PgVector: GIN索引已创建
    end
    
    PgVector-->>User: 优化完成
</mermaid>

## HNSW 索引创建流程

```mermaid
flowchart TD
    A[开始创建HNSW索引] --> B[生成索引名称]
    B --> C[确定距离操作符]
    C --> D[设置索引参数]
    D --> E[构建SQL语句]
    E --> F[执行创建索引]
    F --> G[索引创建完成]
    
    subgraph 索引参数
    D1[m: 每个节点的最大连接数]
    D2[ef_construction: 构建时考虑的候选数]
    D3[ef_search: 搜索时考虑的候选数]
    end
    
    D --> D1
    D --> D2
    D --> D3
</mermaid>

### HNSW 索引创建详解

1. **生成索引名称**：如果未提供，则自动生成
2. **确定距离操作符**：根据距离度量选择适当的操作符
   - L2 距离：`vector_l2_ops`
   - 余弦距离：`vector_cosine_ops`
   - 最大内积：`vector_ip_ops`
3. **设置索引参数**：
   - `m`：每个节点的最大连接数（默认 16）
   - `ef_construction`：构建索引时考虑的候选数量（默认 200）
   - `ef_search`：搜索时考虑的候选数量（默认 5）
4. **构建 SQL 语句**：生成创建索引的 SQL 语句
5. **执行创建索引**：在数据库中执行 SQL 语句

## IVFFlat 索引创建流程

```mermaid
flowchart TD
    A[开始创建IVFFlat索引] --> B[生成索引名称]
    B --> C[确定距离操作符]
    C --> D[设置索引参数]
    D --> E[构建SQL语句]
    E --> F[执行创建索引]
    F --> G[索引创建完成]
    
    subgraph 索引参数
    D1[lists: 聚类中心数量]
    D2[probes: 搜索时检查的聚类数]
    D3[dynamic_lists: 是否动态调整聚类数]
    end
    
    D --> D1
    D --> D2
    D --> D3
</mermaid>

### IVFFlat 索引创建详解

1. **生成索引名称**：如果未提供，则自动生成
2. **确定距离操作符**：根据距离度量选择适当的操作符
3. **设置索引参数**：
   - `lists`：聚类中心数量（默认 100）
   - `probes`：搜索时检查的聚类数量（默认 10）
   - `dynamic_lists`：是否动态调整聚类数量（默认 True）
4. **构建 SQL 语句**：生成创建索引的 SQL 语句
5. **执行创建索引**：在数据库中执行 SQL 语句

## GIN 索引创建流程

```mermaid
flowchart TD
    A[开始创建GIN索引] --> B[生成索引名称]
    B --> C{索引已存在?}
    C -->|是| D{强制重建?}
    D -->|是| E[删除现有索引]
    D -->|否| F[跳过创建]
    C -->|否| G[构建SQL语句]
    E --> G
    G --> H[执行创建索引]
    H --> I[索引创建完成]
    F --> I
</mermaid>

### GIN 索引创建详解

1. **生成索引名称**：通常为 `{table_name}_gin_index`
2. **检查索引存在性**：确认索引是否已存在
3. **处理强制重建**：如果需要强制重建，则删除现有索引
4. **构建 SQL 语句**：生成创建 GIN 索引的 SQL 语句
5. **执行创建索引**：在数据库中执行 SQL 语句

## 索引性能比较

```mermaid
graph LR
    A[索引性能比较] --> B[构建速度]
    A --> C[搜索速度]
    A --> D[内存占用]
    A --> E[准确性]
    
    B --> B1[IVFFlat > HNSW]
    C --> C1[HNSW > IVFFlat]
    D --> D1[IVFFlat < HNSW]
    E --> E1[HNSW > IVFFlat]
</mermaid>

### 索引选择指南

- **HNSW 适用场景**：
  - 需要高准确性的搜索
  - 可以接受较高的内存占用
  - 搜索速度是首要考虑因素
  
- **IVFFlat 适用场景**：
  - 大规模数据集
  - 内存资源有限
  - 构建速度是首要考虑因素
  - 可以接受略低的搜索准确性 