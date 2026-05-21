## rag-laravel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This App Does

A fully local RAG (Retrieval-Augmented Generation) system. Users upload PDF, TXT, or Markdown files; the app extracts text, chunks it, embeds it via a local LM Studio model, and stores vectors in PostgreSQL with pgvector. A chat interface streams answers from a local LLM grounded in the uploaded documents.

## Requirements

- PHP 8.4, PostgreSQL + pgvector, Node.js
- **LM Studio** running locally on port 1234 with two models loaded:
  - Embedding: `text-embedding-nomic-embed-text-v1.5` (768 dimensions — fixed in schema)
  - Chat: `google/gemma-3-4b`

## Commands

```bash
# Start all services (server + queue + logs + Vite HMR)
composer run dev

# Run tests
php artisan test --compact
php artisan test --compact --filter=TestName

# Lint PHP
vendor/bin/pint --dirty --format agent

# Frontend
npm run dev          # Vite dev server
npm run build        # Production build
npm run lint         # ESLint fix
npm run types:check  # TypeScript check
```

> The queue worker (`php artisan queue:listen`) is **required** — document processing runs as a background job. Without it, uploads stay `pending`.

## Architecture

### Document Ingestion Pipeline

Upload → `DocumentController@store` → dispatches `ProcessDocumentJob` → extracts text (PDF via `spatie/pdf-to-text`, MD strips syntax, TXT as-is) → `TextCleaner` → `TextChunker` → `EmbeddingService` (calls LM Studio) → stores `DocumentChunk` rows with `vector(768)` embeddings in PostgreSQL.

`DocumentStatus` enum: `Pending → Processing → Completed | Failed`

### Query & Answer (Streaming SSE)

`POST /rag/stream` → `RagController@stream` → embeds question via `EmbeddingService` → `RagSearchService` performs HNSW cosine search (`whereVectorSimilarTo`, min similarity 0.4, top 8 chunks) → builds context string → streams response via `Laravel\Ai\AnonymousAgent` using SSE (`text/event-stream`).

The stream emits:
1. `event: sources` — JSON array of matching chunk metadata
2. `data: {"delta": "..."}` — token-by-token text
3. `data: [DONE]`

> For production with concurrent users, use Laravel Octane + Swoole. PHP-FPM blocks one worker per open SSE connection.

### Key Services

| Class | Role |
|---|---|
| `EmbeddingService` | Wraps `Laravel\Ai\Embeddings` for single and batch embedding |
| `RagSearchService` | pgvector similarity search, returns `DocumentChunk` collection with `distance` |
| `RagAnswerService` | Non-streaming answer variant (used by `POST /rag/ask`) |
| `TextChunker` | Splits cleaned text into overlapping chunks |
| `TextCleaner` | Normalises whitespace, removes noise |
| `ProcessDocumentJob` | Orchestrates the full ingestion pipeline, retries=3, timeout=300s |

### Frontend

React + Inertia v3. Pages live in `resources/js/pages/`. Shared UI primitives are in `resources/js/components/ui/` (shadcn/Radix). Wayfinder-generated route helpers live in `resources/js/actions/` (controllers) and `resources/js/routes/` (named routes) — import from there instead of hardcoding URLs.

The chat page (`resources/js/pages/chat.tsx`) reads the SSE stream and renders markdown (with KaTeX and Mermaid support).

### Database

PostgreSQL with the `vector` extension. The `document_chunks.embedding` column is `vector(768)` — **this dimension is fixed**. Changing embedding models requires a migration. Uses HNSW index for approximate nearest-neighbour search.

## Environment Variables (key ones)

```env
LMSTUDIO_EMBEDDING_MODEL=text-embedding-nomic-embed-text-v1.5
CHAT_MODEL=google/gemma-3-4b
LMSTUDIO_BASE_URL=http://127.0.0.1:1234/v1
AI_PROVIDER=openai
AI_EMBEDDING_PROVIDER=lmstudio
```

---
> Source: [saturngod/rag-laravel](https://github.com/saturngod/rag-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
