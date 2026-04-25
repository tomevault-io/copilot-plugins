## sqlite-v-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) Vector Server that provides vector similarity search capabilities using SQLite and sqlite-vec extension. The project implements Japanese document storage with configurable OpenAI embeddings and vector search functionality through MCP tools.

## Development Commands

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Development server with hot reload
npm run dev

# Run tests
npm run test
npm run test:watch    # Watch mode
npm run test:coverage # Generate coverage report

# Run migrations
npm run migrate

# Lint and format code
npm run lint
npm run format

# Start server modes
npm start -- --stdio      # stdio mode for MCP protocol
npm start -- --http --port 3000  # HTTP mode with JSON-RPC

# Example scripts
npm run example:direct   # Test MCP direct usage
npm run example:client   # Test MCP client
npm run example:server   # Test server functionality
```

## Architecture

### Core Components

1. **Text Processing Pipeline**:
   - Chunking: 700-character chunks with 100-character overlap for Japanese text
   - Embedding: OpenAI text-embedding-3-small (1536 dimensions) with batch processing
   - Storage: SQLite with sqlite-vec extension for vector operations

2. **MCP Tools**:
   - `insert_document`: Add documents with metadata and generate embeddings
   - `find_similar_documents`: Vector similarity search with cosine distance
   - `delete_document`: Remove documents and their chunks

3. **Server Modes**:
   - **stdio mode**: Direct MCP protocol communication for AI agents
   - **HTTP mode**: Fastify server with `/rpc` endpoint for JSON-RPC 2.0

### Database Schema

```sql
-- Virtual table for vector storage
CREATE VIRTUAL TABLE chunks USING vec0(
  embedding FLOAT[1536]
);

-- Metadata table for document information
CREATE TABLE chunk_metadata (
  chunk_id     TEXT PRIMARY KEY,     -- ULID
  doc_id       TEXT NOT NULL,
  chunk_index  INTEGER NOT NULL,
  text         TEXT NOT NULL,
  metadata     JSON,
  created_at   DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_chunk_metadata_doc_id ON chunk_metadata(doc_id);
```

### Directory Structure

```
src/
├── config/          # Configuration and environment
├── db/              # Database connection and migrations
├── modules/         # Core functionality (chunker, embedder)
├── server/          # Server entry points (stdio/http)
├── tools/           # MCP tool implementations
└── utils/           # Shared utilities
```

### Key Technical Stack

- **sqlite-vec 0.1.x**: Modern vector extension for SQLite
- **TypeScript**: With strict mode enabled
- **better-sqlite3**: Synchronous SQLite bindings
- **Fastify**: HTTP server framework
- **Pino**: Structured JSON logging
- **ULID**: Sortable unique identifiers
- **MCP SDK**: Model Context Protocol implementation
- **OpenAI SDK**: For configurable embedding models

### Development Notes

- WAL mode is enabled for SQLite to support concurrent reads
- Embeddings are configurable (model, dimensions, batch size)
- All async operations use proper error handling
- Database path defaults to `./data/vectors.db`
- OpenAI API key required in `OPENAI_API_KEY` environment variable
- Performance target: P95 < 300ms for 10k chunks on M1 Mac
- Embedding configuration is validated on server startup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Karube) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
