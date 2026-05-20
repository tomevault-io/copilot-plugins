## stratsquad

> > Multi-agent game-industry strategy research console. LangGraph orchestrates a

# StratSquad

> Multi-agent game-industry strategy research console. LangGraph orchestrates a
> 7-agent pipeline (orchestrator + 4 expert sub-agents + judge + composer) against
> DeepSeek V4, BGE-M3 hybrid RAG, BGE reranker, and 9 live trend data sources.
> Next.js + Tailwind frontend, FastAPI + Python backend.

See parent workspace `../CLAUDE.md` for shared context, glossary, and house style (no em dashes, etc.).

## Architecture

```
┌──────────────────────────────┐         ┌───────────────────────────────┐
│  Vercel · Next.js frontend   │         │  Python FastAPI · LangGraph   │
│  app/                        │ ──SSE→  │  server/                      │
│   ├─ page.tsx                │  proxy  │   └─ stratsquad/              │
│   ├─ api/run/route.ts        │         │       ├─ graph.py (StateGraph)│
│   ├─ api/kb/ingest/route.ts  │         │       ├─ nodes/  (7 agents)   │
│   └─ lib/ (types + icons)    │         │       ├─ rag/    (BGE-M3)     │
└──────────────────────────────┘         │       └─ trends/ (9 sources)  │
                                          └───────────────────────────────┘
```

The Next.js routes are thin SSE proxies; **all orchestration runs in Python**.

## Pipeline

```
Strategy question
  → Orchestrator: decompose into 4 sub-briefs (DeepSeek JSON-mode)
  → ┌─ Retrieve: BGE-M3 hybrid (corpus + user KB) → BGE-reranker → top-5 hits
    └─ Trend dispatch: LLM planner picks 4-7 of 9 sources → asyncio.gather
  → Competitor / Trend / Market / Risk: 4 expert agents in parallel
                                        (trend agent cites RAG hits + trend data)
  → Judge: 4-dim rubric (evidence/logic/actionability/novelty), threshold 70
  → conditional retry edge: any verdict='retry' → rerun failed experts → re-judge
  → Composer: integrate 4 outputs into a 战略简报.md
```

LangGraph emits custom events at every step (`agent_start`, `agent_token`, `plan`,
`rag_hits`, `trend_plan`, `trend_result`, `judge`, `brief`, ...). FastAPI forwards
these as SSE; the frontend renders them live.

## Live trend data sources

| Source | Needs key | What it returns |
|---|---|---|
| google-trends | no | Interest-over-time + related queries for 1-5 keywords in a region |
| steam | no (key optional) | Current player counts; Top 10 most-played or specific titles |
| twitch | `TWITCH_CLIENT_ID` + `TWITCH_CLIENT_SECRET` | Top game viewership / streams per game |
| reddit | `REDDIT_CLIENT_ID` + `REDDIT_CLIENT_SECRET` + `REDDIT_USER_AGENT` | 30-day top posts per subreddit + aggregate scores |
| youtube | `YOUTUBE_API_KEY` | Top videos for keywords in a region, with view + like counts |
| appstore | no | iTunes RSS top-free + top-grossing games per country (15 countries supported) |
| huya | no | Mainland China livestream category viewer counts |
| douyu | no | Mainland China livestream rank list per category |
| bilibili | no | Either live-area aggregates (game categories) or video search top results |

All sources gracefully degrade. Missing key or 4xx/5xx response → trend agent still
runs without that source.

## File layout

```
StratSquad/
├── app/                         Next.js App Router
│   ├── page.tsx                 single-page UI (Track 2 FUI design)
│   ├── api/run/route.ts         proxy → Python /api/run
│   └── api/kb/ingest/route.ts   proxy → Python /api/kb/ingest
├── lib/                         FRONTEND ONLY (types + icons)
│   ├── types.ts                 StreamEvent / AgentName / JudgeScore / ...
│   ├── trends/types.ts          TrendSource / TrendQuery / TrendResult / labels
│   ├── rag/types.ts             RagHit / UserChunk
│   └── icons/brands.tsx         9 brand SVG icons
├── server/                      Python LangGraph backend
│   ├── pyproject.toml           uv-managed deps (fastapi, langgraph, langchain, ...)
│   ├── stratsquad/
│   │   ├── main.py              FastAPI app
│   │   ├── graph.py             StateGraph definition
│   │   ├── state.py             StratSquadState + reducers
│   │   ├── agent_runtime.py     LangGraph-aware run_streamed / run_json
│   │   ├── llm.py               DeepSeek via ChatOpenAI
│   │   ├── sse.py               SSE helpers
│   │   ├── types.py             pydantic mirrors of frontend types
│   │   ├── nodes/               7 graph nodes
│   │   ├── rag/                 chunk + embed + retrieve + rerank + store
│   │   └── trends/              9 source clients + planner + dispatch
│   ├── corpus/                  static knowledge base (markdown)
│   ├── data/                    chunks.json + embeddings.json
│   └── eval/                    labeled retrieval test set
├── Dockerfile                   frontend container (Next.js standalone)
└── server/Dockerfile            backend container (Python 3.12 + uv)
```

## Local dev

Two processes:

```bash
# Terminal 1 · Python backend
cd server
uv sync
cp .env.example .env   # fill in DEEPSEEK + SILICONFLOW + trend keys
uv run uvicorn stratsquad.main:app --reload --port 8000

# Terminal 2 · Next.js frontend
npm install
npm run dev            # http://localhost:3002
```

## Key conventions

- **All DeepSeek + LLM logic lives in Python.** Frontend never holds API keys.
- **SSE wire format is preserved** from the old TS pipeline so the UI didn't need
  to change — events still carry `{ type: 'agent_token', agent, delta }` etc.
- **Pydantic models use camelCase aliases** (`alias_generator` in `types.py`) so
  TypeScript can consume JSON without renaming.
- **LangSmith tracing optional.** Set `LANGSMITH_API_KEY` → every run lands in
  the `stratsquad` project with full token-level traces.

## Iteration ideas

- DPO pair generation: rerun the same question with two prompt variants, judge
  picks winner → JSONL ready for DPO fine-tuning.
- Tool-using trend agent: give the experts an actual tool-call surface
  (LangChain `Tool` interface) rather than the pre-fetched bundle.
- Per-session checkpointing: persist `StratSquadState` via LangGraph's checkpoint
  saver, resume across page reloads.
- Cost dashboard: integrate LangSmith → Grafana for per-run DeepSeek spend.

---
> Source: [ChenxingJi-Innovate/stratsquad](https://github.com/ChenxingJi-Innovate/stratsquad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
