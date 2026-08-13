## legal-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Legal AI Research Assistant — a RAG-powered application for legal case research. Python/FastAPI backend with LangGraph RAG pipeline, Next.js 16 frontend with Shadcn/ui components.

## Commands

### Backend
```bash
# Activate virtualenv (Windows Git Bash)
source venv/Scripts/activate

# Install dependencies
pip install -r backend/requirements.txt

# Ingest mock data into Pinecone
python -m backend.ingestion.ingest

# Run backend (hot reload)
python -m uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend
```bash
cd frontend
npm install
npm run dev      # Dev server on :3000
npm run build    # Production build
npm run lint     # ESLint
```

### Quick Start (Windows)
`start-dev.bat` or `start-dev.ps1` launches both servers.

## Architecture

### Backend (FastAPI + LangGraph)

The core is a 6-node LangGraph RAG pipeline in `backend/services/rag_pipeline.py`:

1. **rewrite_question** — reformulates follow-ups as standalone queries
2. **retrieve_documents** — Pinecone vector search (top-5, Voyage voyage-3.5 1024d)
3. **assess_retrieval** — evaluates retrieval confidence from similarity scores
4. **generate_answer** — LLM generates answer (Anthropic Claude Haiku 4.5 primary, Mistral fallback)
5. **self_assess_llm** — LLM evaluates its own confidence
6. **combine_and_score** — weighted confidence: 60% retrieval + 40% LLM self-assessment

Key backend files:
- `backend/main.py` — FastAPI app, CORS config
- `backend/config.py` — Pydantic Settings, all env vars
- `backend/routes/chat.py` — POST `/chat` endpoint (exposes `retrieval_confidence` and `llm_confidence`)
- `backend/services/llm_provider.py` — abstract LLM interface with Anthropic/Mistral implementations
- `backend/services/retriever.py` — Pinecone vector DB wrapper (Voyage query embeddings)
- `backend/services/confidence.py` — dual-layer confidence scoring
- `backend/ingestion/ingest.py` — data loading pipeline (30 synthetic cases → 150 chunks)

### Frontend (Next.js 16 App Router + TypeScript strict)

Chat-first layout with slide-out source drawer. Desktop-optimized.

- `frontend/app/page.tsx` — main page, state management (sessionId, messages, drawer, loading)
- `frontend/app/layout.tsx` — root layout with ThemeProvider and Toaster
- `frontend/app/globals.css` — CSS variables for light/dark themes, animations

Key components:
- `frontend/components/ChatArea.tsx` — chat interface with centered content (max-w-3xl), input bar, scroll-to-bottom
- `frontend/components/Header.tsx` — app header with export, clear, GitHub/LinkedIn links, theme toggle
- `frontend/components/MessageBubble.tsx` — message cards with typewriter effect, copy, confidence badges
- `frontend/components/SourceDrawer.tsx` — inline slide-out drawer (480px) for citations and retrieved sources
- `frontend/components/EmptyState.tsx` — hero with clickable example questions
- `frontend/components/PipelineStepper.tsx` — 5-step horizontal pipeline animation during loading
- `frontend/components/ConfidenceBadge.tsx` — color-coded badge with popover breakdown (retrieval vs LLM)
- `frontend/components/CitationCard.tsx` — expandable citation with court, date, excerpt
- `frontend/components/SourceViewer.tsx` — retrieved chunk cards with match scores

Hooks:
- `frontend/hooks/useTypewriter.ts` — character-by-character text animation with skip and reduced-motion support

Libraries:
- `frontend/lib/api.ts` — API client (`sendMessage()`)
- `frontend/lib/constants.ts` — typewriter speed, pipeline steps, drawer width, message length
- `frontend/lib/utils.ts` — `cn()` utility for classnames
- `frontend/types/index.ts` — TypeScript interfaces (Message, Citation, ChatRequest/Response, RetrievedChunk)

Styling: Tailwind CSS v3 with HSL CSS variables for dark/light themes. Shadcn/ui components in `frontend/components/ui/`.

### Data Flow

Frontend sends full conversation history with each request (stateless backend). Session IDs are UUIDs for logging only. The backend returns answer + citations + confidence scores (retrieval + LLM breakdown) + retrieved chunks.

### Confidence Thresholds

high ≥ 0.50, medium ≥ 0.38, low ≥ 0.25, insufficient < 0.25. Color-coded green/amber/red in UI. (Thresholds calibrated to voyage-3.5 similarity scores, which run lower than OpenAI's.)

## Environment Variables

Backend requires `ANTHROPIC_API_KEY` (LLM), `VOYAGE_API_KEY` (embeddings), and `PINECONE_API_KEY`. Optional: `MISTRAL_API_KEY` for fallback LLM. See `backend/.env.example` for full list. Frontend needs `NEXT_PUBLIC_API_URL` (defaults to `http://localhost:8000`).

## Key Design Decisions

- **Chat-first layout** — full-width chat with slide-out drawer instead of fixed split-panel
- **Inline drawer (not portal)** — drawer is a flex child that transitions width; chat area shrinks naturally
- **Typewriter effect** — only on latest assistant message; respects prefers-reduced-motion
- **Pipeline stepper** — 5 simulated steps during loading (500ms each), completes on API response
- **Dual confidence display** — badge shows overall level; popover shows retrieval/LLM/weighted breakdown
- **Client-side state only** — no backend session storage; conversation history lives in React state
- **LLM fallback** — if Claude fails, automatically tries Mistral (embeddings have no fallback — Voyage only)
- **LangGraph over LangChain chains** — composable, debuggable workflow nodes for multi-step RAG
- **Pinecone free tier** — 100K vector limit; current dataset uses ~150 vectors

## Deployment

- **Frontend:** Vercel (auto-deploys from `main`) — Root Directory set to `frontend` in Vercel dashboard, `frontend/vercel.json` forces Next.js detection
- **Backend:** Railway (auto-deploys from `main`) — configured via `railway.json` + `Procfile`
- **CI:** GitHub Actions in `.github/workflows/` — `frontend-ci.yml` (build check) and `backend-ci.yml` (import check)
- CORS accepts comma-separated origins in `FRONTEND_URL` env var for production
- **Live:** https://legal-ai-hazel-xi.vercel.app (frontend) + https://legal-ai.up.railway.app (backend)

---
> Source: [pradyten/legal-ai](https://github.com/pradyten/legal-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
