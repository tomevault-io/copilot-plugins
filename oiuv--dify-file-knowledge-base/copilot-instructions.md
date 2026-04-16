## dify-file-knowledge-base

> **核心目标**: 本项目旨在创建一个通用的后端服务，该服务遵循 Dify.ai 的外部知识库API规范。它能够索引本地文件目录，并作为一个或多个独立的知识库，通过API响应Dify平台的检索请求。

# Gemini Project Memory: Dify 通用外部知识库服务

## 1. 项目概述

**核心目标**: 本项目旨在创建一个通用的后端服务，该服务遵循 Dify.ai 的外部知识库API规范。它能够索引本地文件目录，并作为一个或多个独立的知识库，通过API响应Dify平台的检索请求。

**项目状态**: 已完成。服务功能完整，采用了“召回+排序”的先进检索架构，并实现了对多个独立知识库的通用化支持。

## 2. 核心架构

项目采用“离线构建、在线服务”的分离式架构。

*   **在线 API 服务 (`main.py`)**: 一个基于 **FastAPI** 的Web服务。它负责实现 Dify 规范的 `POST /retrieval` 接口，处理认证、接收查询、执行检索并返回格式化结果。
*   **离线索引脚本 (`generate_index.py`)**: 一个独立的Python脚本。它负责扫描、解析、向量化源文档，并构建可供快速检索的知识库索引文件。

### 多知识库支持

本项目的核心特性是支持多个独立的知识库。

*   **知识库定义**: `knowledge_base/` 目录下的每一个子目录都被视为一个独立的知识库。该目录的名称即为 Dify API 请求中的 `knowledge_id`。
*   **数据隔离**: 每个知识库的索引文件和元数据文件都存储在 `data/` 目录下，并以 `knowledge_id` 作为前缀（例如 `data/fluffos_index.faiss` 和 `data/fluffos_meta.db`），实现了完全的数据隔离。

## 3. 索引生成流程 (`generate_index.py`)

该脚本负责将原始文档转化为可检索的知识库。

*   **执行方式**: `python generate_index.py <knowledge_id>` 或 `python generate_index.py --all`。
*   **增量更新**: 脚本通过在SQLite数据库 (`<id>_meta.db`) 中存储每个文件的内容哈希值（SHA256）来实现高效的增量更新。只有新增或内容被修改的文件才会被重新处理，已删除的文件其索引也会被同步清理。
*   **元数据提取**: 
    *   **通用化**: 通过在知识库目录中放置一个可选的 `kb_config.json` 文件，可以为每个知识库定义独特的元数据提取规则。
    *   **默认规则**: 如果 `kb_config.json` 不存在，则采用默认规则（从目录结构和文件名中提取分类和主题）。
    *   **`fluffos` 案例**: `fluffos` 知识库使用了 `kb_config.json` 来精确地从文件路径和Markdown头部（frontmatter）提取分类、子分类和主题。
*   **智能分块 (Chunking)**: 采用混合策略。默认将每个文件视为一个完整的块；但如果文件内容过长（超过Token限制），则会按段落进行智能切分，以保证语义完整性。
*   **向量化与优化**: 
    *   **模型**: 使用千问的 `text-embedding-v4` 模型。
    *   **关键优化**: 为了提升检索相关性，在文本被向量化之前，会向其中注入提取出的元数据（如分类、主题），并通过重复核心主题的方式来增强其在向量空间中的权重。

## 4. API 检索流程 (`main.py`)

API服务实现了先进的 **“召回与排序 (Recall & Re-rank)”** 检索流程，以确保结果的高相关性和高准确性。

1.  **查询扩展 (Query Expansion)**: 收到用户原始查询后，首先调用 `qwen-flash` 模型生成多个关键词更明确的查询变体。
2.  **向量召回 (Recall)**: 使用原始查询和所有扩展查询，对FAISS索引进行多次并行的向量搜索，合并所有结果并按向量相似度分数初步排序，筛选出Top N（例如20个）最广泛相关的候选文档。
3.  **精准排序 (Re-rank)**: 将用户的 **原始查询** 和这N个候选文档的内容，提交给专业的 `gte-rerank-v2` 排序模型，获得一个更精准的相关性顺序。
4.  **结果生成与过滤**: 
    *   严格按照 Re-rank 模型给出的 **顺序** 来排列最终结果。
    *   使用在“向量召回”阶段计算出的、我们自己体系的 **原始分数**，与用户请求中 `score_threshold` 进行比较，以决定文档是否最终被返回。
    *   这种方式结合了两种模型的优点，既保证了排序的精准性，又使用了更可控的分数体系进行阈值过滤。

## 5. 关键技术栈

*   **Web框架**: FastAPI, Uvicorn
*   **AI模型服务**: 阿里云通义千问 DashScope (Embedding, Chat, Re-rank)
*   **向量数据库**: FAISS (本地文件索引)
*   **元数据存储**: SQLite
*   **配置管理**: python-dotenv
*   **数据校验**: Pydantic

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oiuv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
