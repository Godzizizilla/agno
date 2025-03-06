# Reader系统详解

Reader系统负责从各种来源读取文档，并将其转换为Document对象。Agno框架支持多种文档格式，每种格式都有对应的Reader实现。

## Reader类层次结构

```mermaid
classDiagram
    class Reader {
        +bool chunk
        +int chunk_size
        +List[str] separators
        +ChunkingStrategy chunking_strategy
        +read(obj) List[Document]
        +chunk_document(document) List[Document]
    }
    
    Reader <|-- PDFReader
    Reader <|-- TextReader
    Reader <|-- URLReader
    Reader <|-- CSVReader
    Reader <|-- JSONReader
    Reader <|-- DocxReader
    Reader <|-- YouTubeReader
    Reader <|-- WebsiteReader
    Reader <|-- ArxivReader
    
    class PDFReader {
        +read(pdf) List[Document]
    }
    
    class TextReader {
        +read(text) List[Document]
    }
    
    class URLReader {
        +read(url) List[Document]
    }
```

## 读取流程

```mermaid
flowchart TD
    A[开始] --> B[选择合适的Reader]
    B --> C[调用Reader.read方法]
    C --> D{是否需要分块?}
    D -- 是 --> E[调用chunk_document方法]
    D -- 否 --> F[返回Document列表]
    E --> F
    F --> G[结束]
```

## 读取和分块过程

```mermaid
sequenceDiagram
    participant App
    participant Reader
    participant Document
    participant ChunkingStrategy
    
    App->>Reader: 创建Reader实例
    App->>Reader: read(source)
    Reader->>Document: 创建Document实例
    
    alt chunk=True
        Reader->>ChunkingStrategy: chunk(document)
        ChunkingStrategy->>ChunkingStrategy: 执行分块逻辑
        ChunkingStrategy-->>Reader: 返回分块后的Document列表
    else chunk=False
        Reader-->>App: 返回原始Document列表
    end
    
    Reader-->>App: 返回最终Document列表
```

## 支持的文档类型

| Reader类 | 支持的格式 | 描述 |
|---------|-----------|------|
| PDFReader | PDF文件 | 读取本地PDF文件 |
| PDFUrlReader | PDF URL | 从URL读取PDF文件 |
| TextReader | 文本文件 | 读取纯文本文件 |
| URLReader | 网页URL | 读取网页内容 |
| CSVReader | CSV文件 | 读取CSV表格数据 |
| JSONReader | JSON文件 | 读取JSON格式数据 |
| DocxReader | DOCX文件 | 读取Word文档 |
| YouTubeReader | YouTube视频 | 读取YouTube视频字幕 |
| WebsiteReader | 网站 | 爬取整个网站内容 |
| ArxivReader | Arxiv论文 | 读取Arxiv学术论文 | 