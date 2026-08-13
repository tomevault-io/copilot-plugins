## lumen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Lumen** — a personal Summa Theologica study app. Three-panel UI: left tree nav, center text, right AI chat. Backend: PostgreSQL (source of truth for Summa text) + Pinecone hybrid search (dense + sparse BM25) + GPT-4.1.

## Commands

### Frontend (Next.js 16, App Router)
```bash
cd frontend
npm install
npm run dev          # http://localhost:3000
npm run build        # production build check
npm run lint
```

### Backend (FastAPI + asyncpg + Pinecone)
```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000   # http://localhost:8000
```

### Data pipeline (run once, in order)
```bash
cd backend
# 1. Set DATABASE_URL in .env, then:
python -m scripts.import_summa    # scrapes newadvent.org → PostgreSQL (~20-40 min)
# 2. After import completes:
python -m scripts.index_summa     # embeds articles → Pinecone (~60-90 min for ~3500 articles)
```

## Architecture

### Frontend (`frontend/`)
- `app/page.tsx` — three-panel layout, lifted state (`selected`, `searchQuery`, `leftOpen`, `rightOpen`)
- `components/SummaTree.tsx` — left panel; full 512-question Summa tree with inline filter search
- `components/ContentViewer.tsx` — center; calls `GET /api/passages` for both node-click and search
- `components/AIChatPanel.tsx` — right panel; calls `POST /api/query`, prepends `[Viewing: ST I Q.N]` context
- `lib/summa-full.ts` — static Summa structure (all 512 questions, article counts) — no API call
- `app/api/passages/route.ts` — Next.js proxy to backend `GET /passages`
- `app/api/query/route.ts` — Next.js proxy to backend `POST /query`
- Theme: pure monochrome dark (`globals.css` CSS variables), Cardo serif + Inter sans fonts

### Backend (`backend/app/`)
- `main.py` — FastAPI lifespan: connects asyncpg pool, ensures DB schema
- `core/config.py` — reads `.env`: `OPENAI_API_KEY`, `PINECONE_API_KEY`, `PINECONE_INDEX_NAME`, `DATABASE_URL`
- `core/dependencies.py` — singleton clients: asyncpg pool, AsyncOpenAI, Pinecone index
- `services/retrieval.py` — `hybrid_search()`: dense (text-embedding-3-large) + sparse (BM25) via Pinecone
- `services/answer.py` — `generate_answer()`: GPT-4.1 with Aquinas system prompt + passage context
- `repositories/article_repo.py` — exact article lookup from PostgreSQL
- `api/routes/passages.py` — `GET /passages?query=...&top_k=8&alpha=0.7`
- `api/routes/article.py` — `GET /article?part_id=prima-pars&question_n=2&article_n=3`
- `api/routes/query.py` — `POST /query` → hybrid_search → generate_answer

### Data layer
- **PostgreSQL** `summa_articles` table: one row per article, with `body TEXT` (full article text), `pinecone_indexed BOOLEAN`
- **Pinecone** index `lumen-summa`: one vector per article, `metric=dotproduct` (required for hybrid), namespace `summa`
- **`data/bm25_params.json`**: BM25 encoder params fitted on full corpus by `scripts/index_summa.py`, loaded at server startup for sparse query encoding

### Search tuning
`alpha` parameter in `hybrid_search()`: `1.0` = pure semantic, `0.0` = pure keyword, default `0.7`. Adjust per query type in `ContentViewer.tsx` → `GET /api/passages` if needed.

## Key env vars
```
DATABASE_URL=postgresql://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres
DATABASE_SSL=true
OPENAI_API_KEY=sk-...
PINECONE_API_KEY=pcsk_...
PINECONE_INDEX_NAME=lumen-summa
PINECONE_NAMESPACE=summa
```

---
> Source: [F0xhopper/Lumen](https://github.com/F0xhopper/Lumen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
