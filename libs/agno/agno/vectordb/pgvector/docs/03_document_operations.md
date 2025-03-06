# PgVector 文档操作

本文档详细说明 PgVector 类的文档操作流程，包括插入、更新和删除文档。

## 文档插入流程

```mermaid
flowchart TD
    A[开始插入] --> B{批量处理}
    B --> |分批处理| C[处理批次]
    C --> D[获取文档嵌入]
    D --> E{文档已存在?}
    E --> |否| F[插入新文档]
    E --> |是| G[跳过文档]
    F --> H{还有更多批次?}
    G --> H
    H --> |是| C
    H --> |否| I[插入完成]
```

### 插入过程详解

1. **批量处理**：文档按批次处理（默认每批 100 个文档）
2. **获取嵌入**：为每个文档内容生成向量嵌入
3. **检查重复**：检查文档是否已存在（基于 ID 或名称）
4. **插入记录**：将文档信息和向量嵌入插入数据库表
5. **错误处理**：出错时回滚当前批次并记录错误

## 文档更新流程（Upsert）

```mermaid
sequenceDiagram
    participant Client
    participant PgVector
    participant Embedder
    participant Database
    
    Client->>PgVector: upsert(documents)
    
    loop 每个批次
        PgVector->>PgVector: 分批处理文档
        
        loop 每个文档
            PgVector->>Embedder: 获取文档嵌入
            Embedder-->>PgVector: 返回嵌入向量
            
            PgVector->>Database: 执行 upsert 操作
            Note right of Database: 如果存在则更新<br>如果不存在则插入
            Database-->>PgVector: 操作结果
        end
    end
    
    PgVector-->>Client: 更新完成
```

### Upsert 过程详解

1. **检查支持**：确认数据库支持 upsert 操作
2. **批量处理**：文档按批次处理
3. **获取嵌入**：为每个文档内容生成向量嵌入
4. **执行 Upsert**：使用 PostgreSQL 的 `ON CONFLICT DO UPDATE` 语法
5. **应用过滤器**：如果提供了过滤器，则应用到操作中

## 文档删除流程

```mermaid
flowchart TD
    A[开始删除] --> B{提供过滤器?}
    B --> |是| C[构建过滤条件]
    B --> |否| D[删除所有记录]
    C --> E[执行删除操作]
    D --> E
    E --> F[返回删除结果]
```

### 删除过程详解

1. **构建条件**：基于提供的过滤器构建 SQL 删除条件
2. **执行删除**：执行 SQL DELETE 语句
3. **返回结果**：返回是否成功删除记录

## 文档存在性检查

PgVector 提供多种方法检查文档是否存在：

```mermaid
flowchart LR
    A[文档存在性检查] --> B[按文档对象检查]
    A --> C[按名称检查]
    A --> D[按ID检查]
    
    B --> E[doc_exists]
    C --> F[name_exists]
    D --> G[id_exists]
    
    E --> H{数据库查询}
    F --> H
    G --> H
    
    H --> I[返回布尔结果]
```

- `doc_exists(document)`：检查文档对象是否存在
- `name_exists(name)`：检查指定名称的文档是否存在
- `id_exists(id)`：检查指定 ID 的文档是否存在 