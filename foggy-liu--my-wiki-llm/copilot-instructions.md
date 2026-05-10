## my-wiki-llm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Artifact Wiki is an LLM-powered knowledge management system for artifact IP content. It uses FastAPI + Vue 3 with file-based Markdown wiki storage. The system has two user roles: admin (content management via Ingest/Lint) and user (knowledge Q&A via Chat).

## Development Commands

```bash
# Backend
cd backend
pip install -r requirements.txt
python run.py                    # Start API server (http://localhost:8000)

# Frontend
cd frontend
npm install
npm run dev                      # Start dev server (http://localhost:3000)

# API proxy: Vite proxies /api/* → http://localhost:8000
```

## Architecture

```
┌─────────────────────────────────────────┐
│  Frontend (Vue 3 + Vite + TailwindCSS) │
│  /admin/*  - Management UI              │
│  /user/*   - User Q&A UI               │
└─────────────────────────────────────────┘
                     REST API
┌─────────────────────────────────────────┐
│  Backend (FastAPI)                     │
│  /api/admin/*  - Ingest, Lint, Publish │
│  /api/user/*   - Query, History        │
│  /api/auth/*  - JWT Auth              │
└─────────────────────────────────────────┘
                     │
┌─────────────────────────────────────────┐
│  Core Services (app/core/)              │
│  ingest.py   - LLM entity extraction    │
│  query.py   - LLM-assisted Q&A         │
│  lint.py    - Quality/health checks     │
│  publish.py - Export to markdown/HTML  │
└─────────────────────────────────────────┘
                     │
┌─────────────────────────────────────────┐
│  Wiki Storage (filesystem Markdown)     │
│  wiki/raw/  - Source materials (read)  │
│  wiki/wiki/ - Processed pages (write)  │
│    entities/, concepts/, summaries/,    │
│    comparisons/, synthesis/           │
└─────────────────────────────────────────┘
```

## Configuration

Environment variables (in `.env` at project root):
```
LLM_API_KEY=          # DashScope API key (通义千问)
LLM_BASE_URL=         # https://dashscope.aliyuncs.com/compatible-mode/v1
LLM_MODEL=            # qwen3.5-plus
SECRET_KEY=           # JWT secret
```

Config is loaded from `backend/app/config.py` → `PROJECT_ROOT / ".env"`.

## Wiki Conventions (from rules.md)

**Critical**: Query is LLM-assisted semantic retrieval, NOT keyword matching. The correct flow: LLM reads `index.md` → understands structure → selects relevant pages → synthesizes answer.

**Wiki page frontmatter** must include:
- `category`: entities | concepts | summaries | comparisons | synthesis
- `confidence`: 0.0-1.0 (required for quality tracking)
- `status`: active | stale | archived
- `sources`: wikilinks to raw source files `[[raw/path/file]]`

**Wikilinks**: Use `[[Page Title]]` syntax. Ingest auto-creates placeholder pages for missing references.

**Log operations**: All Ingest/Query/Lint/Publish operations must be logged to `wiki/wiki/log.md`.

## Key File Locations

| Purpose | Path |
|---------|------|
| Backend entry | `backend/run.py` |
| API routes | `backend/app/api/{auth,ingest,query,lint,publish}.py` |
| Core logic | `backend/app/core/{ingest,query,lint,publish}.py` |
| LLM service | `backend/app/services/llm.py` |
| DB models | `backend/app/db/models.py` |
| Frontend router | `frontend/src/router/index.js` |
| Auth store | `frontend/src/stores/auth.js` |
| Wiki store | `frontend/src/stores/wiki.js` |
| API client | `frontend/src/api/index.js` |

## Common Issues

- **Python process cache**: When restarting backend, kill all python processes first (`taskkill //F //IM python.exe`) to avoid module cache issues
- **CORS**: Backend allows all origins in dev (`allow_origins=["*"]`)
- **JWT token**: Stored in `localStorage.token`, cleared on logout
- **Ingest timeout**: LLM calls can take >30s, frontend axios timeout is 120s for ingest

---
> Source: [Foggy-liu/my-wiki-llm](https://github.com/Foggy-liu/my-wiki-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
