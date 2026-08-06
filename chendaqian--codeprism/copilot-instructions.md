## codeprism

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

WCF API 语义搜索工具 — 基于向量语义搜索的 WCF ServiceContract 接口发现工具链，供大模型调用以查询"某业务应调用哪个接口/方法"。

## 技术栈

- **语言**: C# / .NET 9.0
- **扫描器**: Microsoft.CodeAnalysis.CSharp (Roslyn)
- **向量化**: 阿里云百炼 MaaS 服务（默认 `text-embedding-v4`，1024 维）；亦可通过 `Provider=Ollama` 切回公司内部 Ollama 服务（bge-m3:latest 等）
- **存储**: PostgreSQL + pgvector 扩展
- **MCP**: ModelContextProtocol SDK

## 常用命令

```bash
# 构建整个解决方案
dotnet build CodePrism.sln

# 扫描 WCF 代码 + 直接入库（全量，每次重算向量，按 team/project/interface_name 做 upsert）
dotnet run --project src/CodePrism.WcfCodeScanner -- <path> --team <name> --project <name> --import

# 仅扫描生成 JSON 快照（不入库）
dotnet run --project src/CodePrism.WcfCodeScanner -- <path> --team <name> --project <name> --output output.json

# 带接口/方法过滤
dotnet run --project src/CodePrism.WcfCodeScanner -- <path> --team <name> --project <name> --interface <name> --method <name> --import

# 启动 MCP Server
dotnet run --project src/CodePrism.McpSearchServer
```

> `--team`、`--project` 必填。`--interface`、`--method` 可选过滤（扫描时跳过不匹配的接口/方法）。
> `--import` 模式下不生成 output.json，只生成 JSON 快照时不入库。
> 入库走全量：每次扫描所有 .cs 文件，按 (team, project, interface_name, method_name) 做 upsert，每次重算向量。

**output.json 结构** (部分字段)：
```json
{
  "interfaceName": "IContactControlProvider",
  "assemblyName": "XXX.ServiceInterface",
  "xmlDocComment": "",
  "methods": [{
    "methodName": "GetContact",
    "xmlDocComment": "根据人获取常用联系人 所属租户...",
    "description": "接口：IContactControlProvider\n方法：GetContact\nHTTP：GET...",
    "httpMethod": "GET",
    "parameters": [{"name": "tenantId", "type": "int", "comment": "租户ID"}]
  }]
}
```
- `assemblyName`: 从源文件路径推断的 .csproj 目录名
- `xmlDocComment`: 源码 `/// <summary>` 注释（若有），已清理 XML 标签

## 项目结构

```
src/
├── CodePrism.WcfCodeScanner/      # WCF 代码扫描器（控制台应用）
│   ├── RoslynAnalyzer.cs           # Roslyn 语法分析核心
│   ├── Scanner.cs                  # 全量扫描
│   └── JsonExporter.cs             # JSON 导入导出
├── CodePrism.WcfVectorStorage/   # 向量存储服务（类库）
│   ├── Sql/init.sql                # 数据库初始化 DDL
│   ├── DatabaseInitializer.cs      # 从嵌入资源读取 init.sql 并执行
│   ├── DatabaseConnection.cs       # NpgsqlDataSource 管理
│   ├── EmbeddingService.cs         # 向量化 API 调用（百炼 / Ollama，Provider 切换）
│   ├── WcfRepository.cs            # 数据读写
│   ├── VectorSearchService.cs      # 向量相似度查询
│   └── ConfigurationLoader.cs      # 配置加载
└── CodePrism.McpSearchServer/ # MCP 搜索服务器（控制台应用）
    └── Tools/SearchWcfApiTool.cs   # search_wcf_api 工具定义
```

## 数据库设计

- **wcf_interfaces**: 接口主表（team, project, interface_name）
- **wcf_methods**: 方法表（含 embedding vector(1024)）

向量索引: HNSW 索引，余弦相似度

初始化脚本: `src/CodePrism.WcfVectorStorage/Sql/init.sql`

## 配置文件

`src/CodePrism.WcfVectorStorage/appsettings.json`:
```json
{
  "PostgreSQL": {
    "ConnectionString": "Host=localhost;Port=5432;Database=wcf_search;Username=postgres;Password=xxx"
  },
  "Embedding": {
    "Provider": "Bailian",
    "BaseUrl": "https://ws-v95m39ne1z898zyj.cn-beijing.maas.aliyuncs.com",
    "Model": "text-embedding-v4",
    "ApiKey": "your-bailian-api-key"
  }
}
```

## 完整流程

1. **初始化数据库**: `psql -U postgres -d wcf_search -f src/CodePrism.WcfVectorStorage/Sql/init.sql`
2. **扫描源码 + 入库**（全量，含向量化）: `dotnet run --project src/CodePrism.WcfCodeScanner -- <path> --team <name> --project <name> --import`
3. **启动 MCP Server**: `dotnet run --project src/CodePrism.McpSearchServer`
4. **测试搜索**: `.\scripts\Query.ps1 -Query "关键词"`

> 多团队共存：分别用不同 `--team`、`--project` 扫不同 repo 各自 `--import`，互不影响。每次扫描全量重算向量，按唯一键 upsert。

## MCP 工具

### search_wcf_api

参数:
- `query` (string, 必需): 搜索查询文本
- `topK` (int, 可选): 返回数量，默认 5
- `team` (string, 可选): 团队过滤
- `httpMethod` (string, 可选): HTTP 动词过滤
- 
## 开发规范

- 向量化文本拼接格式见 `design.md` 中的"向量化文本拼接规则"
- 数据库表结构见 `design.md` 中的"数据库设计"
- 接口规格见 `openspec/changes/wcf-api-semantic-search/specs/`

---
> Source: [Chendaqian/CodePrism](https://github.com/Chendaqian/CodePrism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
