## jubilant-octo-journey

> This file provides guidance to agents when working with code in this repository.

This file provides guidance to agents when working with code in this repository.

## Project Overview

CompetitionTutor: A hybrid RAG intelligent Q&A system for academic competition preparation (ACM, math modeling). Uses vector search (Milvus) + knowledge graph (Neo4j) for retrieval-augmented generation with anti-hallucination guarantees.

## Commands

```bash
# Install dependencies
bun install

# Development (starts all apps)
bun dev
# Web: http://localhost:3000 | API: http://localhost:8080

# Run specific workspace
bun run --filter '@repo/web' dev
bun run --filter '@repo/api' dev

# Build/lint/type-check all workspaces
bun build
bun lint
bun type-check

# Testing (423 tests total)
cd packages/rag && bun test                           # 363 tests
cd packages/database && bun --env-file=../../.env test  # 37 tests (requires Docker)
cd apps/api && bun test                               # 23 tests

# Run all tests
bun --env-file=.env test --recursive

# Start databases (Milvus, Neo4j, PostgreSQL)
docker compose -f infrastructure/docker-compose.yml up -d
```

## Architecture

### Monorepo Structure (Bun Workspaces)

- **apps/web** (`@repo/web`): Next.js 16 frontend with Tailwind CSS
- **apps/api** (`@repo/api`): Hono backend running on Bun
- **packages/database** (`@jubilant/database`): Unified database access layer
- **packages/rag** (`@jubilant/rag`): Hybrid RAG pipeline
- **packages/types** (`@repo/types`): Shared TypeScript types
- **packages/config** (`@repo/config`): Shared ESLint/Prettier/TS configs

### Database Layer (`packages/database`)

`DatabaseManager` class provides unified access to all three databases:
- **Milvus**: Vector similarity search for semantic retrieval
- **Neo4j**: Knowledge graph for entity relationships (prerequisites, comparisons)
- **PostgreSQL**: Application data via Drizzle ORM

```typescript
import { db } from '@jubilant/database';
await db.connect();
// db.milvus, db.neo4j, db.postgres
```

### RAG Pipeline (`packages/rag`)

Hybrid retrieval-augmented generation with content-aware processing:

```typescript
import {
  HybridRetriever,
  ContentAwareChunker,
  createCitations,
  Qwen3Reranker
} from '@jubilant/rag';

// Hybrid retrieval with RRF fusion
const retriever = new HybridRetriever(vectorClient, graphClient);
const results = await retriever.retrieve(query, { topK: 10 });

// Content-aware chunking (preserves code, formulas, tables)
const chunker = new ContentAwareChunker({ minTokens: 512, maxTokens: 1024 });
const chunks = chunker.chunk(document, metadata);

// Reranking with confidence threshold
const reranker = new Qwen3Reranker({ confidenceThreshold: 0.6 });
const ranked = await reranker.rerank(query, documents);
```

### Constitution-Driven Development

`.specify/memory/constitution.md` defines binding architectural principles:
- **Hybrid RAG**: All retrieval MUST use both vector AND graph search in parallel
- **Anti-Hallucination**: All responses must cite source material; no fabricated content
- **Dual Interface**: Student Q&A and teacher analytics are strictly separated
- **Content-Aware Processing**: Code blocks, formulas, and tables must not be fragmented during chunking

## Tech Stack

| Layer | Technology |
|-------|------------|
| Runtime | Bun |
| Frontend | Next.js + Tailwind + shadcn/ui + TanStack Query |
| Backend | Hono + Drizzle ORM |
| LLM | Qwen3-32B (local), Qwen3-Embedding-8B, Qwen3-Reranker-4B |
| RAG Framework | LlamaIndex.TS |
| Vector DB | Milvus |
| Graph DB | Neo4j |
| Relational DB | PostgreSQL |

## Test Coverage

| Package | Tests | Coverage |
|---------|-------|----------|
| `packages/rag` | 363 | Chunking, citations, retrieval, streaming, prompts, error handling |
| `packages/database` | 37 | Milvus, Neo4j, PostgreSQL integration |
| `apps/api` | 23 | Throttle middleware, request handling |

Key test files:
- `packages/rag/tests/unit/` - Unit tests for all RAG components
- `packages/rag/tests/integration/` - Query and ingestion pipeline tests
- `packages/database/tests/integration/` - Database connection tests
- `apps/api/tests/unit/throttle.test.ts` - Concurrency handling

## Environment Variables

Required in `.env`:
```bash
# Database connections
MILVUS_HOST, MILVUS_PORT
NEO4J_URI, NEO4J_USER, NEO4J_PASSWORD
POSTGRES_HOST, POSTGRES_PORT, POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB

# LLM services
LLM_BASE_URL, LLM_MODEL
EMBEDDING_BASE_URL, EMBEDDING_MODEL
RERANKER_BASE_URL, RERANKER_MODEL

# RAG configuration
RAG_VECTOR_TOP_K, RAG_GRAPH_MAX_HOPS, RAG_RERANK_TOP_K
RAG_CONFIDENCE_THRESHOLD, RAG_CHUNK_SIZE
```

## Speckit Workflow

This project uses speckit for feature development. Available commands:
- `/speckit.specify` - Create feature specs
- `/speckit.plan` - Generate implementation plans
- `/speckit.tasks` - Generate task lists
- `/speckit.implement` - Execute implementation
- `/speckit.clarify` - Identify underspecified areas
- `/speckit.analyze` - Cross-artifact consistency check

## Active Technologies
- TypeScript 5.x (Bun runtime) + Next.js 16, Hono, shadcn/ui, Drizzle ORM, Qwen3 LLM (OpenAI-compatible API) (006-chat-conversation-ui)
- PostgreSQL (conversations + query metrics), localStorage (client-side message content), Milvus (vectors), Neo4j (graph) (006-chat-conversation-ui)
- TypeScript 5.x (Bun runtime) + Next.js 16 (frontend), Hono (backend), shadcn/ui (components), Drizzle ORM, TanStack Query (data fetching), `@jubilant/rag` (ETL pipeline) (007-knowledge-etl)
- PostgreSQL (documents, jobs), Milvus (vectors), Neo4j (graph) (007-knowledge-etl)
- TypeScript 5.x (Bun runtime) + Next.js 16 (frontend), Hono (backend), Recharts (charts), shadcn/ui (components), TanStack Query (data fetching), Drizzle ORM (PostgreSQL), Neo4j driver (008-teacher-dashboard)
- PostgreSQL (rag_queries, retrieval_metrics, feedback_events), Neo4j (Concept nodes, DISCUSSES relationships) (008-teacher-dashboard)

## Recent Changes
- 006-chat-conversation-ui: Added TypeScript 5.x (Bun runtime) + Next.js 16, Hono, shadcn/ui, Drizzle ORM, Qwen3 LLM (OpenAI-compatible API)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Qnurye) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
