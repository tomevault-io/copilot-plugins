## odm-aidataplatform-demo

> This file gives coding agents the project context and guardrails for this repository.

# AGENTS.md

This file gives coding agents the project context and guardrails for this repository.

## Project Overview

This project is an ODM AI data center and knowledge cleaning platform.

It turns scattered engineering materials such as Feishu documents, enterprise Wiki pages, defect records, test reports, SOP documents, Android log summaries, and historical issue reviews into reusable data assets for RAG, evaluation sets, bad case feedback, and engineering Agent applications.

## Core Data Flow

```text
External engineering sources
  -> Raw engineering records
  -> Machine filtering and masking
  -> Staging data
  -> Manual review and labeling
  -> Custom knowledge supplements
  -> High-quality engineering knowledge
  -> RAG / evaluation sets / bad case feedback / Agent applications
```

## Core Modules

- Raw engineering data library
- Data governance and staging
- Manual review and labeling
- Custom knowledge
- Knowledge extraction
- Engineering knowledge service desk
- Engineering attachment library
- Newcomer capability assessment
- Data export
- Global search

## Tech Stack

- Frontend: React 18, TypeScript, Vite, TailwindCSS
- Backend: FastAPI, SQLAlchemy, Pydantic
- Database: PostgreSQL with pgvector
- AI: OpenAI-compatible LLM APIs and embedding services
- Deployment: Docker Compose

## Development Commands

Backend:

```bash
cd backend
cp env.example .env
uv sync
uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Frontend:

```bash
cd frontend
npm install
npm run dev
```

Database:

```bash
docker compose -f docker-compose.db.yml up -d
```

Build checks:

```bash
cd frontend
npm run build
```

```bash
python3 -m compileall backend/app backend/scripts scripts
```

## Development Rules

- Do not commit `.env`.
- Do not expose real company data, internal links, defect IDs, logs, or API keys.
- Use mock ODM engineering data for demos.
- Keep startup commands and public API routes stable unless a task explicitly asks for a deeper migration.
- Do not rename database tables or compatibility fields casually; many routes and tests still depend on the existing schema.
- Keep user-facing language aligned with ODM engineering knowledge governance.

## Public Demo Notes

The public demo intentionally reuses some legacy schema names and API paths to avoid risky migrations. Business-facing pages should present the data as raw engineering records, staging data, high-quality knowledge, and downstream export assets.

---
> Source: [helloaiden0305/odm_AIDataPlatform_demo](https://github.com/helloaiden0305/odm_AIDataPlatform_demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
