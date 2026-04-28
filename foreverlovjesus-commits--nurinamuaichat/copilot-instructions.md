## nurinamuaichat

> **NuriNamu AI Chat** is a production-grade RAG chatbot for Korean public-sector

# NuriNamu AI Chat — Project Instructions for Claude

## Project Identity

**NuriNamu AI Chat** is a production-grade RAG chatbot for Korean public-sector
compliance (청탁금지법/이해충돌방지법/행정심판법 등). Built to meet 조달청 납품 수준의
보안·감사 요구사항.

Not a generic "PDF chatbot". Domain-specific legal Q&A with FIRAC-style reasoning,
hybrid retrieval, metadata filtering, and audit logging.

## Stack

- **Backend**: Python 3.11, FastAPI, asyncpg, LangChain
- **Frontend**: Next.js 14+ (App Router), TypeScript, React
- **Admin**: Streamlit (13 pages — dashboard/debug/ontology/prompts/billing)
- **Vector DB**: PostgreSQL + pgvector (local), Supabase (production)
- **Embeddings**: BAAI/bge-m3 (local CPU) — 1024-dim multilingual
- **LLM**: Gemini 2.5 Flash (primary), OpenAI/Anthropic/Ollama (alternate providers)
- **Reranker**: BAAI/bge-reranker-v2-m3 (cross-encoder)

## Architecture Principles

1. **Factory pattern for retriever**: `retriever/factory.py` returns the active
   retriever based on `RETRIEVER_TYPE` env var. Never import concrete classes
   (`AdvancedHybridRetrieverV2`) directly outside the factory.

2. **Hybrid retrieval is the default**: Vector search + Postgres FTS + RRF fusion.
   Metadata filtering uses **adaptive multi-level fallback** (strict → loose → none).

3. **Metadata tagging is critical**: `indexer/metadata_tagger.py` uses LLM +
   rule-based fallback. Rule-based MUST run independently of LLM success
   (past incident: LLM timeout zeroed out all filters).

4. **Content-type routing**: `detect_doc_type()` in `indexer/rag_indexer.py`
   decides `legal|case|faq|general` based on folder + filename. MD files in
   `/faq/` folder must route to `general` (past incident: pandas parsing crash).

5. **Security-first**: API_KEY header auth, rate limiting (slowapi), PII masking
   (`server/input_guard.py`), audit logging (`server/audit_logger.py`),
   Fernet-encrypted DATABASE_URL.

6. **Audit every /ask**: Success AND failure must reach `log_audit_event`.
   Silent drop of audit is a compliance violation for 조달청 납품.

## Repository Layout

```
rag/              RAGEngineV3 — prompt assembly, LLM routing, streaming
retriever/        factory + AdvancedHybridRetrieverV2 (vector+FTS+RRF+rerank)
indexer/          rag_indexer, metadata_tagger, pdf_parser, law_graph, law_scheduler
integrations/     MCP law client (국가법령정보센터), context merger
server/           FastAPI (api_server.py), db_manager, audit, input_guard
admin_pages/      Streamlit pages (01_monitor ~ 13_notebook_chat)
frontend/         Next.js chat UI
monitoring/       query_logger
scripts/          helper .bat scripts (Windows-friendly)
doc_archive/      source documents: 법령/판례/FAQ
docs/             operator manuals + deployment guides
```

Empty-but-reserved: `auth/`, `web/` (legacy static UI is in `static/`).

## Environment & Configuration

All runtime config via `.env` at project root. Key variables:
- `MAIN_LLM_MODEL`, `GLOBAL_LLM_PROVIDER`, `GEMINI_MODEL`
- `TAGGING_LLM_PROVIDER`, `TAGGING_CONCURRENCY`, `TAGGING_DELAY`
- `RETRIEVER_TYPE`, `GLOBAL_EMBEDDING_MODEL`
- `API_KEY`, `MASTER_KEY`, `ENCRYPTED_DATABASE_URL`
- `ENABLE_FIRAC_MODE`, `FIRAC_FORMAT_STYLE`, `WEIGHT_CASE`, `WEIGHT_LEGAL`
- `HIDE_SESSION_LIST`, `HIDE_COMPARE_TAB` (UI feature flags, read by `/config/ui`)

Never hardcode secrets. `.env` must NOT be committed or copied into Docker images.

## Platform Notes

Primary dev environment: **Windows 11** (Korean locale).
- `PYTHONUTF8=1` required for asyncpg + 한글 경로
- `PGPASSFILE` must be `NUL` on Windows, `/dev/null` on Linux
- Use forward-slash paths even in bash tool
- Use PowerShell `Start-Process -WindowStyle Hidden` for background tasks
  (bash `&` kills children on shell exit)

## Claude's Role

Act as **Senior Code Reviewer** for this project.

Responsibilities:
- Detect logic errors, race conditions, compliance gaps
- Validate architecture against the principles above
- Suggest improvements with concrete patches
- Flag Windows/Linux portability issues aggressively

Rules:
1. **Do not automatically modify files.** Present issues first, wait for approval.
2. **Never create alternate versions** (`*_v2.py`, `*_new.md`). Small, targeted edits only.
3. **Change format**: Issue → Impact → Proposed Fix → Files Affected
4. Respect the factory pattern — don't bypass `get_retriever()`.
5. When in doubt about a production behavior, check `server/api_server.py` first
   (it's the source of truth for request flow).
6. Hot-reload / global mutation inside request handlers is a known anti-pattern
   here. Flag it, don't replicate it.

## Known Pitfalls (learned the hard way)

- **Gemini response parsing**: Gemini may return `content` as `list[dict]`, not
  `str`. Always extract `'text'` field from dict items before JSON-parsing.
- **LLM tagging timeout**: Local Ollama needs ~60s timeout, API LLMs ~10s.
  Rule-based fallback must run OUTSIDE the try/except.
- **Rate limiting Gemini**: Start at `TAGGING_CONCURRENCY=5, TAGGING_DELAY=1.0`.
  Concurrency=10 triggers 503 UNAVAILABLE within ~100 requests.
- **Multi-worker state**: Do NOT mutate `global_retriever` inside `/ask`.
  Uvicorn `--workers N` runs N processes — mutations don't propagate.
- **Preview LLM models**: Never use `*-preview` models in production — no SLA.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/foreverlovjesus-commits) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
