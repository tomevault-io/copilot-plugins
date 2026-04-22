## alfinresearchscraperglobal

> Full-stack research discovery engine: searches for academic/policy PDFs, downloads them, analyses content via LLM, and classifies source domains. Built for finding and curating research papers on specific topics from reputable institutions.

# CLAUDE.md — Autonomous Research Engine

## Project Overview

Full-stack research discovery engine: searches for academic/policy PDFs, downloads them, analyses content via LLM, and classifies source domains. Built for finding and curating research papers on specific topics from reputable institutions.

## Architecture

```
Frontend (React + Vite + Tailwind v4)  →  Backend (FastAPI + SQLite)  →  LLMs (Gemini/OpenRouter)
     :5173                                    :8000                        Serper (search API)
```

- **Backend**: FastAPI app in `backend/`, Python 3.12, venv at `backend/venv/`
- **Frontend**: React 19 + Vite 8 + Tailwind CSS v4 in `frontend/`
- **Database**: SQLite with WAL mode (`backend/research_pipeline.db`)
- **PDF Storage**: `backend/data/` (flat directory, gitignored)

## Key Files

### Backend (`backend/`)
| File | Purpose |
|------|---------|
| `api.py` | FastAPI routes — all endpoints under `/api/` |
| `db.py` | SQLite schema, UPSERT helpers, domain seeding. Tables: `pdf_files`, `domains` |
| `expansion.py` | LLM query expansion (topic → 5 web + 5 scholar queries) via Instructor |
| `search.py` | Serper API integration (web + scholar search) |
| `downloader.py` | PDF download with PRIMP (15s) → Botasaurus fallback (30s). Per-domain method memory |
| `analyser.py` | PDF analysis via LLM — extracts summary, type, usefulness, domain reputation |
| `domains.py` | Domain reputation (DB-only lookups, no LLM). Updated during PDF analysis |
| `discovery_loop.py` | Iterative search loop until target PDF count reached |
| `extractor.py` | PDF text extraction via PyMuPDF |
| `llm.py` | LLM client setup (Vertex AI / OpenRouter fallback) |

### Frontend (`frontend/src/`)
| File | Purpose |
|------|---------|
| `App.jsx` | Main app — 3-tab layout (Search, Downloads, Domains), search loop, analysis orchestration |
| `api.js` | Axios API client — all backend calls |
| `components/DownloadsTable.jsx` | Downloads grid — filters, sorting, resizable columns, bulk actions, retry/upload |
| `components/DomainsPanel.jsx` | Domain reputation management panel |

## Dev Environment

### Start Backend
```bash
cd backend
source venv/bin/activate    # MUST use venv — primp, botasaurus, etc. are installed here
uvicorn api:app --host 0.0.0.0 --port 8000 --reload
```

### Start Frontend
```bash
cd frontend
npm run dev    # Vite dev server on :5173
```

### Required Environment Variables (`backend/.env`)
```
OPENROUTER_API_KEY=...        # Primary LLM provider
SERPER_API_KEY=...            # Search API
USE_VERTEX_AI=true/false      # Optional: use Google Vertex AI instead
GCP_PROJECT_ID=...            # If Vertex AI enabled
GCP_LOCATION=...              # If Vertex AI enabled
MODEL_QUERY_EXPANSION=...     # Model for query expansion (default: gemini-2.0-flash-lite)
```

### Install Dependencies
```bash
# Backend
cd backend && python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
pip install python-multipart primp botasaurus

# Frontend
cd frontend && npm install
```

## Database Schema

Two tables in `research_pipeline.db`:

**`pdf_files`** — one row per PDF (PRIMARY KEY: `filename`)
- Download fields: `filename`, `filepath`, `url`, `domain`, `title`, `query_used`, `size_bytes`, `download_status`
- Analysis fields: `document_title`, `summary`, `document_type`, `useful_for_research`, `usefulness_reasoning`, `publication_date`, `publisher`, `analysis_status`

**`domains`** — domain reputation cache (PRIMARY KEY: `domain`)
- `verdict`: `reputable` | `not_reputable` | `unknown`
- `download_method`: `primp` | `botasaurus` (learned per-domain)
- `topics`: JSON array of research topics
- 40 seed domains pre-loaded (BIS, IMF, arxiv, etc.)

## API Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/expand` | Semantic query expansion |
| POST | `/api/discover/web` | Web search via Serper |
| POST | `/api/discover/scholar` | Scholar search via Serper |
| POST | `/api/download-all` | Download PDFs (PRIMP → Botasaurus) |
| POST | `/api/retry-download` | Retry failed downloads via Botasaurus |
| POST | `/api/analyse-stream` | SSE streaming PDF analysis |
| POST | `/api/upload` | Upload PDFs (with optional `replace_url` to update failed row) |
| POST | `/api/classify-domains` | Domain lookup (DB-only, no LLM) |
| GET | `/api/downloads` | List all PDFs (auto-reconciles orphans from disk) |
| GET | `/api/domains` | List all classified domains |
| DELETE | `/api/files` | Delete files from disk + DB |
| POST | `/api/copy-to-research` | Copy files to `data/research/` |
| GET | `/api/export` | JSON backup of all DB data |

## Patterns & Conventions

- **DB writes**: Use UPSERT pattern (`INSERT ... ON CONFLICT DO UPDATE`). Each function opens its own connection for thread safety.
- **Downloads**: PRIMP first (15s timeout, Chrome impersonation), Botasaurus fallback (30s). Domain preference is remembered in `domains.download_method`.
- **Analysis**: SSE streaming — each PDF result sent as `data:` event, final summary as `event: done`.
- **Domain reputation**: No separate LLM call. Assessed during PDF analysis based on actual document quality. Seeds provide initial reputable list.
- **Query dedup**: Used queries persisted per topic in localStorage. LLM is told to avoid repeating them.
- **Frontend state**: `mergedFiles` is the primary downloads state. `topic` persisted in localStorage.
- **File uploads**: When uploading against a failed download row, pass `replace_url` to update the existing DB row instead of creating a new one.

## Common Tasks

- **Purge all data**: Delete `backend/research_pipeline.db` and `backend/data/*.pdf`, then restart backend (calls `init_db()` on startup).
- **Re-run project analysis**: `python analyse_project.py` from repo root — updates `project_structure.md`.
- **Add a new API endpoint**: Add route in `api.py`, add Pydantic model if needed, add API function in `frontend/src/api.js`, wire into `App.jsx`.

## Things to Watch

- Backend MUST run from the venv (`backend/venv/bin/uvicorn`), not system Python
- `python-multipart` is required for file upload endpoints
- SQLite uses WAL mode — safe for concurrent reads but only one writer at a time
- Botasaurus requires Chrome/Chromium installed on the system
- Frontend API base URL is hardcoded to `http://localhost:8000/api` in `api.js`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/madhuys) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
