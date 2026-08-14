## ai-study-hub-rag-service

> Guide for AI assistants working in `ai-study-hub-rag-service`. Everything below is grounded in the current source tree.

# Repository Guidelines

Guide for AI assistants working in `ai-study-hub-rag-service`. Everything below is grounded in the current source tree.

## Project Overview

**AI Study Hub RAG Service** — a FastAPI / Python service that powers the AI features of the AI Study Hub platform. It owns document **ingestion** (download → parse → parent-child chunk → embed → index), **hybrid retrieval** (BM25 + pgvector dense + Jina cross-encoder re-rank), and **RAG generation** (Gemini) with numeric `[N]` citations. It stores vectors in PostgreSQL + pgvector and shares one `aistudyhub` database (plus one `INTERNAL_API_SECRET`) with the sibling **Java backend** (`ai-study-hub-api`). Runs on uvicorn, port `8000`.

## Architecture & Data Flow

Single FastAPI app (`main.py`) exposing REST endpoints. There is no DB ORM layer — persistence is raw `psycopg2` over a process-wide `ThreadedConnectionPool` (`app/database/pool.py`). External LLM/embedding/rerank clients are process-wide singletons (`app/core/clients.py`), warmed at startup.

```
Java backend ──HTTP──▶ FastAPI (main.py)
                          │
   ingest endpoints       │           chat endpoint: POST /api/v1/chat
   POST /rag/extract      │           │
   POST /rag/index        │           ├─ guardrail (runs BEFORE router, in main.chat_router):
   PATCH /rag/.../visibility│         │     • validate_input + detect_prompt_injection (always ON)
   DELETE /rag/documents/{id}│        │     • check_policy_topic (LLM) only if ENABLE_POLICY_GUARDRAIL=1
                          │           │     • block ─▶ HTTP 200 canned refusal (no retrieval/generation)
                          │           ▼
                          └─▶ route_chat_request (deterministic: SMALLTALK → SUMMARY → QA)
                                        QA branch: retrieve_documents()
                                        ├─ BM25 (parent docs, filtered by document_id)
                                        ├─ dense pgvector cosine (HNSW, k=25)
                                        ├─ EnsembleRetriever (BM25 0.3 / dense 0.7)
                                        ├─ Jina re-rank → top context
                                        └─ Gemini generation (+ history) → [N] citations
   ◀── callback (X-Internal-Secret) ─ send_callback() ──▶ backend
```

**Observability.** Every `/chat`, `/quiz/generate`, `/flashcard/generate`, and ingestion BackgroundTask opens a **Langfuse** root trace — `trace_chat` / `trace_material` / `trace_ingest` in `app/core/langfuse_client.py` — with sub-stages wrapped in `lf_span(name)` and LLM calls auto-captured via LangChain callbacks. All tracing is **fail-open at runtime** (`LANGFUSE_ENABLED=0`, missing keys, or any SDK error → no-op); the request path is never affected. See *Code Conventions → Instrumentation* for the helper inventory + per-route metadata.

**Ingestion is two-phase** (`app/services/ingestion.py`):
- `_extract`: download presigned file → load by extension → enrich metadata (page/chunk citations + `document_id`) → parent-child chunk via `ParentDocumentRetriever._split_docs_for_adding` → store parent docs in `LocalFileStore` (`parent_docs_store/`) → insert child chunks into `document_chunks` with **`embedding = NULL`**.
- `_index`: `embed_pending_chunks(document_id)` — embed all `embedding IS NULL` chunks (one `embed_documents` call, 1536-dim Gemini) + per-row `UPDATE` → `update_bm25()`. Idempotent.
- `process_document_task` (PRIVATE docs) = `_extract` + `_index` + summary, then callback `SUCCESS`. `extract_document_task` (PUBLIC docs) = `_extract` only + summary, callback `EXTRACTED`. `index_document_task` (after approval) = `_index`, callback `SUCCESS`.

Callbacks (`send_callback`) POST to `BACKEND_CALLBACK_URL` (`/api/v1/internal/documents/callback`) with header `X-Internal-Secret`, body `{document_id, status, summary}`, retried 3× with exponential backoff.

## Key Directories

```
main.py                     FastAPI app: endpoints, request models, startup warmup
app/
├── core/
│   ├── config.py           Settings (DATABASE_URL, BACKEND_CALLBACK_URL, INTERNAL_API_SECRET, LANGFUSE_*, ...)
│   ├── clients.py          Singleton Gemini LLM + Jina reranker (process-wide)
│   ├── performance.py      Instrumentation: start_trace() / stage() / trace.emit() → logs/performance.log
│   └── langfuse_client.py  Langfuse SDK v4 singleton + fail-open helpers: get_langfuse / get_langchain_callbacks
│                           / lf_span(name) / trace_chat / trace_material / trace_ingest
├── database/
│   ├── pool.py             ThreadedConnectionPool + db_connection() (rollback on exit, test-on-borrow SELECT 1, TCP keepalives)
│   ├── vector_store.py     Custom PostgresVectorStore over pgvector (add_texts, embed_pending_chunks,
│   │                       delete_by_document_id, update_chunk_visibility, similarity_search_by_vector)
│   └── document_store.py   document lookups (summary, title, user document ids)
├── services/
│   ├── ingestion.py        _extract / _index / process / extract / index tasks + delete + visibility
│   ├── retrieval.py        Hybrid retrieval (BM25+dense) + Jina re-rank; optional follow-up query-rewrite
│   ├── guardrail.py       /chat input guardrail: validate_input + detect_prompt_injection (always ON) + check_policy_topic (LLM, ENABLE_POLICY_GUARDRAIL); block → HTTP 200 canned refusal
│   ├── router.py           Deterministic router (regex): SMALLTALK / SUMMARY / QA — no LLM
│   └── generation.py       Gemini RAG answer with [N] citations; consumes history (multi-turn)
└── pipeline/
    └── dependencies.py     Singletons: embeddings, vectorstore, parent/child splitters,
                            ParentDocumentRetriever, LocalFileStore docstore, BM25 state (initialize/update_bm25)
parent_docs_store/          Persistent LocalFileStore for parent docs (keyed by uuid) — mounted volume
temp/                       Downloaded source files (cleaned up after ingest) — mounted volume
logs/                       performance.log (RotatingFileHandler) — mounted volume
initdb.sql                  Full DDL (shared with backend): document_chunks.embedding vector(1536) + HNSW cosine
requirements.txt            Pinned deps — LangChain 0.3.x (do NOT bump to 1.x)
Dockerfile / docker-compose.yml   python:3.11-slim image; joins external ai-study-hub-network
```

## Development Commands

Python + uvicorn. **There is no Java/Node here — this is a pure Python service.**

```bash
# Local dev (from the repo root) — reload on change
uvicorn main:app --reload --host 0.0.0.0 --port 8000
# or
python main.py

# Install deps (use a virtualenv)
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Run the whole stack via Docker (builds image, serves :8000)
docker compose up --build -d
```

- **Needs a running PostgreSQL 16 + pgvector** (`aistudyhub` DB, `document_chunks` table with `vector(1536)` + HNSW index) — normally provided by the backend's `docker-compose.yaml`. `initialize_bm25()` at startup reads parent docs from `parent_docs_store/`; if empty, BM25 starts empty (added as docs are indexed).
- **Needs env / API keys** (see Runtime below). Copy them into `.env` — `load_dotenv()` loads it.
- **Swagger**: `http://localhost:8000/docs`.

## Code Conventions & Common Patterns

**Singletons.** External clients (`llm`, `embeddings`, `reranker`) and pipeline objects (`vectorstore`, `store`, `retriever`, splitters, `state`) are created **once at import** in `app/core/clients.py` and `app/pipeline/dependencies.py`, then imported everywhere. Never instantiate per-request — they hold HTTP connection pools / TLS state (a former per-request `new ChatGoogleGenerativeAI(...)` cost ~50-150ms of setup each). `_warmup_clients()` at startup primes Gemini (avoids a ~14s cold-start on the first real request).

**DB access.** Always borrow from the pool via the `db_connection()` context manager (`app/database/pool.py`) — it `rollback()`s in `finally` (returns the connection clean to the pool, even for reads) and is closed by `atexit`. `ThreadedConnectionPool` is used because handlers are sync `def` (see below). `minconn=1`, `maxconn=DB_POOL_MAX` (default 20). Connections are opened with **TCP keepalives + a connect timeout** and validated on checkout (cheap `SELECT 1`); a connection that died idle (e.g. postgres restarted while it sat in the pool) is discarded (`putconn(close=True)`) and replaced once before yielding — so a PG restart no longer bricks this service (root cause of the 2026-07 upload-FAILED incident, where a stale pool surfaced as `server closed the connection unexpectedly` → FAILED callbacks to the Java backend). If the DB is genuinely down, the second connection surfaces the error to the caller (no infinite retry).

**Sync handlers.** Endpoints use plain `def` (not `async def`) — FastAPI runs them in a threadpool, so blocking I/O (Gemini, Jina, psycopg2) does not stall the event loop. Keep new endpoints `def` unless they're genuinely async.

**Two-phase extract/index (moderation gate).** Public docs are extracted with `embedding = NULL` and only embedded after the backend approves them. `similarity_search_by_vector` filters `WHERE embedding IS NOT NULL`, so extracted-but-not-indexed chunks are never returned. Never remove that filter.

**Parent-child retrieval.** `_extract` reuses `ParentDocumentRetriever._split_docs_for_adding` so child chunks carry `metadata["doc_id"]` = parent uuid (LangChain `MultiVectorRetriever.id_key = "doc_id"`). Retrieval matches a child by vector, then fetches its parent from `LocalFileStore`. Do not change the `doc_id` key without updating both sides. Splitters: parent 1000/200, child 200/50.

**Instrumentation (dual: perf log + Langfuse).** Two parallel systems, both must stay fail-safe:
- *Local perf log* — wrap request work in `start_trace(label, **meta)` and sub-steps in `with stage("name"):`, emitted to `logs/performance.log` + console (`ENABLE_PERF_LOG=1` default).
- *Langfuse tracing* (`app/core/langfuse_client.py`) — a **fail-open** singleton (`get_langfuse()` → `None` when disabled / missing keys / SDK error) plus helpers: `lf_span(name)` (used **alongside** `stage(name)` for the same step, so both perf log + Langfuse get it), `get_langchain_callbacks()` (passed as `config={"callbacks": ...}` so Gemini calls auto-report input/output/usage/cost/latency), and root-trace contexts `trace_chat` / `trace_material` / `trace_ingest`. Pipeline functions are decorated with `@observe` — `retrieve_documents`, `generate_rag_response`, `generate_quiz`, `generate_flashcards` — so they become child observations of the request root trace.
- *Per-route metadata* on the root trace: `/chat` records `route` ∈ {`guardrail_block`, `smalltalk`, `summary`, `qa`, `error`, `exception`} + `empty_retrieval` / `retrieved_count` / `refusal_category` / `error`; the QA branch additionally emits a `citation_coverage` **score** (0..1 = valid `[N]` markers ÷ retrieved docs). Quiz/flashcard traces record `route` ∈ {`quiz`, `flashcard`} + `refused` / `generated`. Ingestion traces record `status` (SUCCESS/EXTRACTED/FAILED) + `chunks_embedded` / `summary_len`.
- *Import-safety caveat:* fail-open covers **runtime** only (disabled flag, missing keys, network/SDK errors). It does **not** cover the top-level `from langfuse import observe` in `retrieval.py` / `generation.py` / `study_material.py` — langfuse must be installed **and** importable (Python ≥3.10, see Runtime) or the app will not start. Never wrap new tracing in bare `try/except` at import time; use the helper functions which already fail-open.
The codebase labels perf optimizations as `S2` (singletons), `S3` (sync handlers/threadpool), `S4` (connection pool), `S6` (BM25 pre-filter), `S8` (multi-query off) — match these markers when extending. **Never** let a Langfuse error escape into the request path.

**Error → callback.** Ingestion tasks catch exceptions and `send_callback(document_id, "FAILED")` rather than raising (they run in `BackgroundTasks`, off the request thread). Always send a terminal callback so the backend's status machine doesn't stall.

**Callbacks to backend.** `send_callback` uses stdlib `urllib.request` (not `requests`/`httpx`) with the `X-Internal-Secret` header; 3 retries, exponential backoff.

**Intent routing (deterministic, no LLM).** `route_chat_request()` classifies each `/chat` query by regex into SMALLTALK (greetings/thanks/farewell) → SUMMARY (explicit summary request on a selected `document_id`) → QA (default). The router makes **no LLM call** — a 3-way intent split is trivially rule-based; an LLM here only added latency + a quota increment per request. `document_id == null` is always QA (SUMMARY needs a specific doc). Misses are safe: a paraphrased summary request with no keyword falls through to QA, which still answers over the selected doc.

**Input guardrail (`/chat` only).** `check_chat_request()` (`app/services/guardrail.py`) runs in `main.chat_router` **before** `route_chat_request`, over `query` + `history`. Three branches, first block wins; every block returns **HTTP 200** with a canned refusal in `data.llm_response` (+ `data.debug.guardrail{category,reason}`) — no retrieval/generation: (1) `validate_input` — deterministic, always ON (empty / over-length query, control & zero-width chars except `\n`/`\t`, history >10 turns or malformed items); (2) `detect_prompt_injection` — EN+VI rule-based regex, always ON (override/extraction, role-hijack, chat-template injection), scans `query` **and** each `history` content; (3) `check_policy_topic` — LLM classifier, **OFF by default** (`ENABLE_POLICY_GUARDRAIL=1`), fail-open on any LLM/parse error → ALLOW. Refusal locale follows `query.isascii()` (ASCII→EN, else VI), matching `_smalltalk_reply`. Uploaded documents are **not** scanned here (moderated upstream at ingestion). Constants `MAX_QUERY_LENGTH=2000`, `MAX_HISTORY_TURNS=10`, `MAX_HISTORY_ITEM_LENGTH=2000` live in the module (tuning params, not env).

**Multi-turn memory.** The backend sends the session's prior turns as `history` (`{role, content}`, oldest first, capped at 10) in the `/chat` body. `generate_rag_response()` injects them as a "Conversation so far" block so the model can resolve follow-up references (pronouns, "that", "as you mentioned"). History is **auxiliary only** — the answer must still be grounded in the retrieved context and cite `[N]` from it, never from history. For context-dependent follow-ups (e.g. "hãy trả lời nội dung đó"), `ENABLE_QUERY_REWRITE=1` rewrites the query into a self-contained one (1 LLM call) and feeds it to retrieval + rerank **only** — the generator still receives the original query (option b). Default OFF; triggered only when the query looks like a follow-up (pronouns/deictics) and history is present.

**Empty-retrieval guard.** When the QA branch retrieves zero relevant chunks, `chat_router` short-circuits with a fixed "no information" message instead of calling the generator on an empty context (avoids hallucination + saves a generation call).

**Quiz & flashcard generation (`/quiz/generate`, `/flashcard/generate`).** Unlike `/chat` (query-scoped top-k retrieval), generation works over a document's **full content in reading-order** (`get_document_content` → `chunk_index ASC`, capped ~30k chars) — quizzes/flashcards need broad, coherent coverage. Output is **structured JSON**, enforced via `llm.with_structured_output(pydantic_schema)` (Gemini native JSON mode: `response_mime_type=application/json` + `response_schema`), not a free-text string the prompt merely "asks" to be JSON — schema-validated at parse time, with one retry on a parse failure. **Two-layer refusal** (first wins): (1) deterministic content floor (too short → refuse pre-LLM, no Gemini call); (2) an LLM `suitable` flag (fragmented / non-textual content sets `suitable=false`). A refusal returns `GenerationResult(refused=True)` → HTTP 200 with empty items + `data.debug.refused` (NOT an error status), mirroring the guardrail canned-refusal pattern. Language follows the document's language (the `CRITICAL LANGUAGE RULE` proven by `generate_document_summary`). The optional `focus` param is the only user free-text → guarded by `detect_prompt_injection` only (no full guardrail; no `/chat`-style `query`).

## Important Files

| File | Purpose |
|------|---------|
| `main.py` | FastAPI app: endpoints (`/rag/process`, `/rag/extract`, `/rag/index`, `/rag/documents/{id}/visibility`, `/rag/documents/{id}`, `/chat`, `/quiz/generate`, `/flashcard/generate`), request models (`ChatRequest` carries `history` for multi-turn memory; `StudyMaterialRequest` for quiz/flashcard), startup (BM25 init + client warmup). `/chat` runs the input **guardrail** (`check_chat_request`) before the router; a block returns HTTP 200 with a canned refusal + `data.debug.guardrail`. Each `/chat` & `/quiz`/`/flashcard` request opens a Langfuse root trace (`trace_chat`/`trace_material`) and records per-route metadata + a `citation_coverage` score via `_lf_record`/`_lf_record_material` (fail-open). The old `/chat/retrieve` debug endpoint was removed (it searched the whole store unfiltered). |
| `app/services/ingestion.py` | Two-phase `_extract`/`_index` + `process/extract/index_document_task`, `delete_document`, `update_document_visibility`, `send_callback`, `generate_document_summary`. Each BackgroundTask wraps its body in a Langfuse `trace_ingest` root (`ingest-process`/`ingest-extract`/`ingest-index`) with `lf_span` per stage + records terminal `status` |
| `app/services/retrieval.py` | `retrieve_documents(query, document_ids, history=None)` — hybrid BM25+dense (`EnsembleRetriever`) + Jina rerank; BM25 pre-filtered by `document_id`; optional follow-up query-rewrite (`ENABLE_QUERY_REWRITE` — retrieval/rerank only, generator keeps the original query). Decorated `@observe(name="retrieval-funnel")`; stages use `lf_span` + LangChain callbacks (top-level `from langfuse import observe`) |
| `app/services/router.py` | `route_chat_request()` — deterministic regex router: SMALLTALK → SUMMARY (needs `document_id`) → QA (default); **no LLM**. Returns `smalltalk` / `summary` / `qa` / `error` |
| `app/services/generation.py` | `generate_rag_response(query, documents, history=None)` — Gemini answer with `[N]` citation markers; injects prior turns (`history`) to resolve follow-up references. Decorated `@observe(name="rag-generation", as_type="span")`; LLM call passes `get_langchain_callbacks()` (top-level `from langfuse import observe`) |
| `app/services/study_material.py` | `generate_quiz(document_id, count, focus)` / `generate_flashcards(...)` — structured-output generation via `llm.with_structured_output(...)` (Gemini native JSON mode). Works over the document's **full content in reading-order** (not query-scoped retrieval like `/chat`). Two-layer refusal: deterministic content floor (pre-LLM, no quota cost) + an LLM `suitable` flag (fragmented/non-textual). Refusal → HTTP 200 + empty items + `data.debug.refused`. Returns `GenerationResult{items, refused, reason}`. Both generators are `@observe`-decorated (`quiz-generation`/`flashcard-generation`); structured-output invoke passes `get_langchain_callbacks()` (top-level `from langfuse import observe`) |
| `app/services/guardrail.py` | `check_chat_request(query, history)` — input guardrail run before the router: `validate_input` (deterministic) + `detect_prompt_injection` (EN+VI regex) always ON; `check_policy_topic` (Gemini classifier) opt-in via `ENABLE_POLICY_GUARDRAIL`, fail-open. Block → HTTP 200 canned refusal (locale via `query.isascii()`); returns `GuardrailResult{allowed, refusal, category, reason}` |
| `app/database/vector_store.py` | Custom `PostgresVectorStore`: `add_texts` (embed), `add_texts_without_embedding` (extract), `embed_pending_chunks` (index), `delete_by_document_id`, `update_chunk_visibility`, `similarity_search_by_vector` (`embedding IS NOT NULL`) |
| `app/pipeline/dependencies.py` | Pipeline singletons: `embeddings` (Gemini 1536-dim), `vectorstore`, `store` (LocalFileStore), `retriever` (ParentDocumentRetriever), splitters, BM25 `state` + `initialize_bm25`/`update_bm25` |
| `app/database/document_store.py` | `get_document_summary` / `get_document_title` / `get_user_document_ids` / `get_document_content(document_id, max_chars)` — concatenates a document's chunks in reading-order (`chunk_index ASC`, `embedding IS NOT NULL`) for quiz/flashcard generation |
| `app/core/clients.py` | Singleton `llm` (`gemini-2.5-flash-lite`) + `reranker` (`jina-reranker-v3`, top_n=5) |
| `app/core/config.py` | `Settings`: `DATABASE_URL`, `BACKEND_CALLBACK_URL`, `INTERNAL_API_SECRET`, `TEMP_DIR`, `ENABLE_POLICY_GUARDRAIL` (default `0`), `LANGFUSE_ENABLED` (default `1`), `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`, `LANGFUSE_BASE_URL` (default `https://cloud.langfuse.com`) |
| `app/core/performance.py` | `start_trace` / `stage` / `PerformanceTrace.emit` → `logs/performance.log` (`ENABLE_PERF_LOG`) |
| `app/core/langfuse_client.py` | Langfuse SDK v4 singleton + fail-open helpers: `get_langfuse()`, `get_langchain_handler()`/`get_langchain_callbacks()` (LangChain `CallbackHandler` to capture LLM usage/cost/latency), `lf_span(name)` (span, pairs with `stage()`), root-trace contexts `trace_chat`/`trace_material`/`trace_ingest`. All return `None`/no-op when `LANGFUSE_ENABLED=0`, keys missing, or SDK error — runtime never affected. Lazy-imports `langfuse` inside helpers EXCEPT the service modules' top-level `from langfuse import observe` (must be installed: Python ≥3.10) |
| `initdb.sql` | Shared DDL: `document_chunks(id, document_id, chunk_index, content, embedding vector(1536), metadata jsonb, page_number)` + `CREATE EXTENSION vector` + HNSW `vector_cosine_ops` index |
| `requirements.txt` | Pinned: `fastapi`, `uvicorn`, **`langchain>=0.3,<0.4` (pinned — 1.x breaks)**, `langchain-google-genai`, `psycopg2-binary`, `rank_bm25`, `pypdf`, `docx2txt`, `python-dotenv`, **`langfuse>=4.7,<5` (needs Python ≥3.10)** |
| `Dockerfile` / `docker-compose.yml` | `python:3.11-slim`; container `ai-study-hub-rag-service` on external `ai-study-hub-network`; mounts `parent_docs_store/`, `temp/`, `logs/` |
| `docs/langfuse-metrics-cookbook.md` | Contract doc (NOT runtime) for the **Java backend** to call Langfuse **Metrics API v2** (`GET {LANGFUSE_BASE_URL}/api/public/v2/metrics?query=...`, Basic auth) for an admin dashboard — per-stage latency p95, token/cost by model, `citation_coverage` avg, refusal/empty-retrieval/route counts. Backend needs its own `LANGFUSE_PUBLIC_KEY`/`LANGFUSE_SECRET_KEY`/`LANGFUSE_BASE_URL` (same Langfuse project as this service) |

## Sibling Service: Backend API (Java)

The Java backend (`~/code/ai-study-hub-api`, Spring Boot 4.0.6) is the API gateway and owns `documents`/`users`/`chat_sessions`/etc. This RAG service **owns `document_chunks`** (writes embeddings + metadata). The two share one PostgreSQL `aistudyhub` DB and one `INTERNAL_API_SECRET`.

**Contract (this service's side):**
- Receives `POST /api/v1/rag/process` (private: extract+index), `/extract` (public: extract only), `/index` (after approval: embed pending), `PATCH /rag/documents/{id}/visibility`, `DELETE /rag/documents/{id}`, `POST /api/v1/chat` — all from the backend over the shared network. (The old `/chat/retrieve` debug endpoint was removed.)
- `/chat` body: `{query, user_id, document_id, history}` — `history` is the session's prior turns (`{role, content}`, oldest first, ≤10) for multi-turn memory. The input **guardrail** (`check_chat_request`) runs first; a block returns HTTP 200 with a canned refusal in `data.llm_response` (+ `data.debug.guardrail{category,reason}`) and skips retrieval/generation. Otherwise the router internally picks SMALLTALK (canned reply) / SUMMARY (precomputed) / QA (retrieval + generation, with an empty-retrieval guard). Response shape is unchanged (`data.llm_response` + `data.debug`).
- `/quiz/generate` & `/flashcard/generate` body: `{document_id, count, focus}` — document-scoped study-material generation (`count` clamped 5–20 quiz / 5–30 flashcard; optional `focus` topic, injection-guarded). Unlike `/chat`, generation works over the document's **full content** (reading-order chunks), not query-scoped retrieval. Output is structured JSON (`data.quiz[]` = `{question, options[4], correct_index, explanation}`; `data.flashcards[]` = `{term, definition}`), returned in the same `{success, message, data, timestamp}` envelope. The backend (Java `StudyMaterialController` `/api/v1/study-materials/{quiz,flashcard}`) enforces the daily AI quota + document access before calling these.
- Sends `POST` to `${BACKEND_CALLBACK_URL}` (= backend `/api/v1/internal/documents/callback`) with `X-Internal-Secret: ${INTERNAL_API_SECRET}`, body `{document_id, status: SUCCESS|EXTRACTED|FAILED, summary}`.
- **Langfuse admin dashboard (backend-side).** `docs/langfuse-metrics-cookbook.md` specifies how the Java backend queries Langfuse **Metrics API v2** (Basic auth with `LANGFUSE_PUBLIC_KEY`:`LANGFUSE_SECRET_KEY`, `GET {LANGFUSE_BASE_URL}/api/public/v2/metrics?query=<URL-encoded JSON>`) to feed an admin stats page: per-stage latency p95, endpoint volume, token/cost by model, `citation_coverage` avg, refusal/empty-retrieval/route-distribution counts. This is a contract doc — **not** part of this service's runtime. The backend will need its own `LANGFUSE_PUBLIC_KEY`/`LANGFUSE_SECRET_KEY`/`LANGFUSE_BASE_URL` env pointing at the same Langfuse project this service traces to. Note: Langfuse `metadata.*` is filter-only (cannot `GROUP BY`), so route-distribution etc. require N filtered queries or client-side aggregation (see cookbook §3.9).

**Gotchas:**
- `INTERNAL_API_SECRET` here must equal the backend's `app.internal.secret`, else every callback is rejected with 403.
- **The backend reads `document_chunks` read-only** (its `DocumentChunkRepository`) for moderation — it never writes this table. Only this service writes `document_chunks`.
- The backend gates the explicit `document_id` it sends to `/chat` (only `COMPLETED` docs). For the multi-doc path (`document_id == null`), this service resolves the scope itself via `get_user_document_ids`, which now filters `status = 'completed' AND deleted_at IS NULL` — so PENDING/FAILED/REJECTED/soft-DELETED docs are excluded from **both** BM25 and dense retrieval at the source (previously only the dense branch's `embedding IS NOT NULL` safety net applied, leaving a BM25 leak via parent docs of non-indexed docs). Dense search still also filters `embedding IS NOT NULL`.
- **Smalltalk/greetings** (detected in this service's router) return a canned reply with no retrieval and no citations — but the backend still counts them against the daily AI quota, because it increments the counter *before* calling RAG. If chitchat should be free, the backend must detect it itself.
- **Guardrail blocks return HTTP 200, not an error.** A blocked `/chat` query returns `success:true` with a canned refusal in `data.llm_response` (same shape as a normal answer); the only distinguishing signal is `data.debug.guardrail{category,reason}`. The backend cannot tell a guardrail refusal from a real answer by status code — if it must (e.g. to skip the daily AI quota), inspect `data.debug.guardrail` or do its own input check. Validation + injection are always ON; the LLM policy layer is opt-in and fail-open (LLM error → ALLOW), so it never hard-blocks on its own.
- **Quiz/flashcard refusals return HTTP 200 with empty items** (`data.quiz`/`data.flashcards` = `[]` + `data.debug.refused=true`), not an error — same envelope as a success. Two layers: (1) deterministic content floor (doc too short → refuses before calling Gemini); (2) an LLM `suitable` flag (fragmented/non-textual content). The backend detects a refusal via the empty item list and surfaces `data.debug.reason`/`message`. Note: the backend increments the daily AI quota *before* the RAG call, so a refusal still consumes quota (same trade-off as smalltalk).

- See the backend's `AGENTS.md` for the full document lifecycle / moderation flow.

## Runtime / Tooling Preferences

- **Runtime**: Python. **`Dockerfile` targets `python:3.11-slim`** (production). The local `.venv` in this checkout is **Python 3.9** (EOL) — it now **fails to start**: `langfuse>=4.7` requires Python ≥3.10, and `retrieval.py`/`generation.py`/`study_material.py` do a top-level `from langfuse import observe`, so importing them on 3.9 raises `ModuleNotFoundError`/`ImportError` before uvicorn serves a single request. Create a 3.11+ venv locally to match the image (this also removes the Google `FutureWarning`s).
- **Server**: uvicorn (`main:app`). `python main.py` runs uvicorn with `--reload`.
- **Dependencies**: `pip install -r requirements.txt`. **LangChain is pinned to 0.3.x — do not bump to 1.x** (it removed `langchain.storage`, reshuffled `langchain.retrievers`, etc.; this code targets the 0.3 API). ChromaDB is intentionally **not** used — vectors live in pgvector via the custom `PostgresVectorStore`.
- **Container**: `docker-compose.yml` builds `.` and joins the **external** `ai-study-hub-network` (created by the backend's compose). Mounts `parent_docs_store/`, `temp/`, `logs/` so parent docs and logs survive restarts.
- **External APIs**: Google Gemini (LLM `gemini-2.5-flash-lite`, embeddings `gemini-embedding-001` forced to **1536 dims**) + Jina (`jina-reranker-v3`). Keys via env (`GOOGLE_API_KEY` consumed by langchain-google-genai, `JINA_API_KEY`).
- **Env vars** (`.env`, loaded by `dotenv`): `DATABASE_URL`, `BACKEND_CALLBACK_URL`, `INTERNAL_API_SECRET`, `GOOGLE_API_KEY`, `JINA_API_KEY`, `ENABLE_MULTI_QUERY` (default `0` — multi-query costs ~6s/extra LLM call), `ENABLE_QUERY_REWRITE` (default `0` — rewrites context-dependent follow-ups into a self-contained query for retrieval only; ~1 extra LLM call per follow-up), `ENABLE_POLICY_GUARDRAIL` (default `0` — turns ON the `/chat` policy/topic LLM guardrail; the validation + injection layers are always ON regardless), `LANGFUSE_ENABLED` (default `1` — master switch; set `0` to fully disable tracing), `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY` (both required when enabled, else instrumentation no-ops with a warning), `LANGFUSE_BASE_URL` (default `https://cloud.langfuse.com`; self-host v3 URL or regional cloud e.g. `https://jp.cloud.langfuse.com`), `DB_POOL_MAX` (default 20), `ENABLE_PERF_LOG` (default `1`), `TEMP_DIR` (default `temp`). Never commit `.env`.

## Testing & QA

- **No automated tests** — there is no `tests/` directory and no `pytest`/`unittest` in `requirements.txt`. Verification is manual: run uvicorn, hit `/docs`, and exercise an endpoint with a real `document_id`/`file_url`.
- **Performance log**: `logs/performance.log` (rotating) records per-request stage timings via `start_trace`/`stage`/`emit`. Check it to profile retrieval latency (embed_query, dense_sql, bm25_build, rerank, generation).
- **Local sanity checks before changing ingestion/retrieval**: confirm `initialize_bm25` logs the parent-doc count at startup, and that `document_chunks` rows get `embedding` filled after `/index` (a `NULL` after a successful `/index` means `embed_pending_chunks` failed).

---
> Source: [SWP-SUM26-AI-STUDY-HUB/ai-study-hub-rag-service](https://github.com/SWP-SUM26-AI-STUDY-HUB/ai-study-hub-rag-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
