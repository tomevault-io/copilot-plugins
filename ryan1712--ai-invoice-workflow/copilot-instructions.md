## ai-invoice-workflow

> validates extracted data, routes uncertain documents to human review, and exports approved results.

# AI Batch Invoice Processing Workflow System

## What this is

A batch invoice processing system. Users upload multiple invoice PDFs at once.
The system creates a batch, processes each document independently in background workers,
validates extracted data, routes uncertain documents to human review, and exports approved results.

This is a real software system — not a demo app.

## Architecture

See docs/03-architecture.md for full detail.

```
Next.js Frontend
      │ HTTP REST
FastAPI Backend
      ├── BatchService          — batch creation, state derivation, batch export
      ├── DocumentService       — document state transitions, history
      ├── ParseService          — PDF text extraction (PyMuPDF)
      ├── ExtractionService     — Azure OpenAI structured extraction
      ├── ValidationService     — field validation rules
      └── ExportService         — JSON / Excel / CSV generation
      │
      ├── PostgreSQL
      ├── Local File Storage
      ├── Redis                 — Celery task broker
      └── Azure OpenAI

Celery Workers (separate process)
      └── DocumentProcessingTask → Parse → Extract → Validate → Route
```

## Workflow States

Document states:
```
uploaded → parsing → extracting → validating → needs_review / approved → exported / failed
```

Batch state is derived from document states — never stored or set directly.

## Engineering Rules

- Never implement beyond the current milestone scope
- Only DocumentService writes document state — no other service updates state directly
- Batch state is always derived on read by BatchService — never stored
- LLM is called through LLMProvider interface — never call Azure OpenAI directly from routes or tasks
- PDF parsing goes through ParserProvider interface
- All configuration via environment variables — never hardcode credentials or paths
- Monetary and numeric values stored as TEXT in database
- `corrected_value: null` = not edited; `""` = user cleared the field — export layer must distinguish
- `processing_history` is append-only — never update or delete history records
- Line item editing is excluded from initial delivery scope — do not implement
- Retry resets document state to `uploaded` and writes a new history record

## Current Milestone

**Milestone 1 — Project Skeleton**

Scope:
- FastAPI backend skeleton with settings module
- Next.js frontend skeleton
- docker-compose: backend, frontend, worker, redis, db (PostgreSQL)
- Health check endpoint: GET /health → { "status": "ok" }
- README with setup instructions

Do not implement: upload, processing, extraction, validation, review UI, export.

## Stack

- Backend: Python / FastAPI
- Frontend: Next.js
- Database: PostgreSQL
- Task Queue: Celery + Redis
- LLM: Azure OpenAI
- PDF parsing: PyMuPDF
- Deployment: Docker / docker-compose

## Excluded from Initial Delivery Scope

- Authentication
- Multi-user / multi-tenant
- Scanned PDF OCR
- Azure Document Intelligence
- ERP integration
- Semantic search / RAG
- LangGraph
- Automatic retry with back-off
- Line item editing in review UI

## Key Docs

- docs/01-product-brief.md
- docs/02-requirements.md
- docs/03-architecture.md
- docs/04-api-contract.md
- docs/05-data-model.md
- docs/06-decisions.md
- docs/07-tasks.md

---
> Source: [Ryan1712/ai-invoice-workflow](https://github.com/Ryan1712/ai-invoice-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
