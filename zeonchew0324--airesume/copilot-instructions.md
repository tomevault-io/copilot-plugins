## airesume

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Frontend (run from `frontend/`)
```bash
npm run dev      # Dev server on http://localhost:5173
npm run build    # TypeScript + Vite production build
npm run lint     # ESLint
npm run preview  # Preview production build
```

### Backend (run from `backend/`)
```bash
python -m uvicorn app.main:app --reload  # Dev server on http://localhost:8000
pip install -r requirements.txt          # Install dependencies
```

Backend requires a `.env` file in `backend/` with `OPENAI_API_KEY`.

## Architecture

### Data Flow
1. User uploads PDF resume + job title/description on the frontend
2. Frontend POSTs `FormData` to `/api/analyze`
3. Vite dev proxy forwards `/api/*` → `http://localhost:8000`
4. Backend route → service (PDF extraction) → LangChain graph (AI scoring)
5. Returns `ResumeAnalysisResponse`: `match_score`, `feedback`, `suggestions`, `missing_keywords`
6. Frontend renders results in tabbed UI (pie chart score, feedback, suggestions, keywords)

### Backend (`backend/app/`)
- **`main.py`** — FastAPI app with CORS for `localhost:5173`, mounts routers
- **`routes/`** — HTTP layer; delegates to `services/`
- **`services/`** — Business logic; extracts PDF text, calls LangChain graph
- **`graphs/`** — LangGraph workflows; constructs the ChatOpenAI chain with structured output
- **`prompts/`** — System/user prompt templates for ATS evaluation
- **`models/`** — Pydantic request/response schemas
- **`utils/`** — PDF parsing (PyPDF2)

Model used: `gpt-4o-mini` via LangChain-OpenAI.

### Frontend (`frontend/src/`)
- **`App.tsx`** — React Router with two routes: `/analyze` (functional), `/improve` (placeholder)
- **`pages/`** — `AnalyzeResume` page owns the full upload + results flow
- **`components/`** — `AppLayout` (sidebar nav), `ScoreChart` (Recharts PieChart), shadcn UI primitives
- Path alias `@/` maps to `src/`

### Key Tech
- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS 4, shadcn/ui, Recharts
- **Backend**: FastAPI, LangChain + LangGraph, OpenAI, PyPDF2, Pydantic

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zeonchew0324) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
