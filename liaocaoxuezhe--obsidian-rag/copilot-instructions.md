## obsidian-rag

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Obsidian RAG (Retrieval-Augmented Generation) system** that converts Obsidian knowledge bases into intelligent conversation systems. It supports semantic retrieval, local deployment, and incremental updates. The project integrates with Claude Code through MCP (Model Context Protocol) for seamless RAG functionality.

## Key Commands

### Environment Setup

```bash
# Install dependencies and set up virtual environment
./setup.sh

# Activate virtual environment (if not using scripts)
source venv/bin/activate
pip install -r requirements.txt
```

### Core Operations
```bash
# Build vector index (required before first use)
./run.sh build --vault /path/to/obsidian/vault

# Interactive query mode
./run.sh query

# Interactive chat mode
./run.sh chat

# Start web interface
./run.sh web

# Update index incrementally
./run.sh update

# Daemon mode for automatic updates
./run.sh daemon

# Clean all data
./run.sh clean
```

### Testing
```bash
# Basic functionality test
python test_basic.py

# MCP server test
python test_mcp_simple.py
python test_mcp.py
```

## Architecture

### Core Components

1. **Document Processing Pipeline**
   - `src/document_parser.py`: Parses Markdown files, extracts YAML frontmatter, tags, links
   - `src/text_chunker.py`: Intelligent text splitting with multiple strategies (semantic, paragraph, fixed-size)
   - `src/embedding.py`: Vector embedding generation using sentence-transformers

2. **Storage & Retrieval**
   - `src/database.py`: ChromaDB vector database operations
   - `src/retrieval.py`: Multi-strategy retrieval (vector, keyword, hybrid search)

3. **Interface Layers**
   - `src/chat.py`: Multi-turn conversation system with context management
   - `src/query.py`: Direct query interface
   - `src/web_server.py`: FastAPI web interface
   - `src/mcp_server.py`: MCP server for Claude Code integration

4. **Index Management**
   - `src/build_index.py`: Builds initial vector index
   - `src/update_index.py`: Incremental updates and file change monitoring

### Data Flow

```
Obsidian Files → Document Parser → Text Chunker → Embedding Generator → Vector Database
                                                                              ↓
User Query ← Retrieval System ← Chat/Query Interface ← MCP Server/Web Interface
```

## Configuration

### Environment Variables (.env)
- `OBSIDIAN_VAULT_PATH`: Path to Obsidian vault (required)
- `EMBEDDING_MODEL`: Model name (default: paraphrase-multilingual-MiniLM-L12-v2)
- `EMBEDDING_DEVICE`: cpu/mps/cuda (default: cpu)
- `CHROMA_PERSIST_DIRECTORY`: Vector DB path (default: ./data/chroma)
- `TOP_K_RETRIEVAL`: Number of results (default: 5)
- `SIMILARITY_THRESHOLD`: Similarity threshold (default: 0.7)
- `LLM_PROVIDER`: mock/openai/local (default: mock)

### Config File (config.yaml)
Main configuration file with detailed settings for:
- Embedding models and parameters
- Database configuration
- Text chunking strategies
- Retrieval parameters
- Performance optimization

## MCP Integration

The system provides MCP tools for Claude Code:

- `vector_search`: Semantic search in knowledge base
- `rag_chat`: RAG-powered conversation
- `get_document_info`: Document metadata
- `build_index`: Index management
- `search_suggestions`: Query suggestions
- `list_documents`: Document browsing

## Development Notes

### Key Design Patterns
- **Modular Architecture**: Each component is independently testable
- **Configuration-Driven**: All parameters configurable via YAML/environment
- **Incremental Processing**: Supports efficient updates without full rebuilds
- **Multi-language Support**: Optimized for Chinese and English content

### Performance Considerations
- **Batch Processing**: Embedding generation uses batching for efficiency
- **Caching**: Embedding vectors and file hashes are cached
- **GPU Acceleration**: Supports CUDA and Apple Silicon (MPS)
- **Parallel Processing**: Configurable worker processes

### File Structure Importance
- `src/`: Core implementation modules
- `data/`: Vector database and cache storage
- `logs/`: Application logs
- `tests/`: Test files
- `config.yaml`: Main configuration
- `.env`: Environment variables (copy from `.env.example`)

## Common Workflows

1. **Initial Setup**: `./setup.sh` → configure `.env` → `./run.sh build`
2. **Development**: Modify code → `python test_basic.py` → `./run.sh update`
3. **MCP Usage**: Ensure index built → Use Claude Code with MCP tools
4. **Web Interface**: `./run.sh web` → Access http://localhost:8000

## Important Notes

- **UTF-8 Encoding**: All file operations should maintain UTF-8 encoding for Chinese compatibility
- **Index Required**: Must build index before any query/chat operations
- **Virtual Environment**: Always use the provided virtual environment
- **Configuration Priority**: Environment variables override config.yaml settings

---
> Source: [liaocaoxuezhe/obsidian-rag](https://github.com/liaocaoxuezhe/obsidian-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
