## ct-chatbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

RAG chatbot for Corporate Turnaround (business debt-settlement company). LangGraph ReAct agent + hybrid retrieval (Chroma semantic + BM25 keyword, RRF fusion, cross-encoder rerank) over a curated JSON knowledge base, served via Streamlit or a CLI loop.

## Commands

```bash
source venv/bin/activate                      # local venv at ./venv

# Ingest KB into Chroma (required before first run, and after ANY KB edit)
python scripts/ingest.py --loader enriched --force

# Run
uvicorn api.main:app --port 8000              # HTTP API (production; backs the website widget)
python warmup.py                              # Streamlit UI, retrieval warmed before the port opens
python main.py                                # CLI chat loop

# Tests / eval
pytest tests/                                 # unit tests (guardrails)
python scripts/smoke_test.py                  # end-to-end smoke (uses LLM API quota)
python scripts/eval_retrieval.py              # retrieval eval vs data/eval_retrieval_set.json
python scripts/diagnose_retrieval.py "query"  # inspect what retrieval returns for one query

# KB rebuild (raw sources → enriched_knowledge_base.json)
python scripts/build_kb_v2.py
```

Run pytest with a single test: `pytest tests/test_guardrails.py -k <name>`.

## Critical rules

- **Editing anything under `data/` changes nothing at runtime until you re-ingest** (`python scripts/ingest.py --loader enriched --force`). The agent reads only from `./chroma_db`.
- Gemini free-tier API quota is limited — don't loop LLM-calling scripts (smoke_test, synthesize_qa_answers) unnecessarily. Retrieval-only scripts (eval, diagnose) use no LLM quota.
- Secrets live in `.env` (`GEMINI_API_KEY`, optionally `GROQ_API_KEY`); `.env.example` documents them.
- The PII phone allowlist in `config.py` exists because Presidio otherwise redacts CT's own published phone numbers — don't remove it. Ingest-time PII scrubbing is deliberately off (see comments in `PIIConfig`).

## Architecture

Two independent phases: **ingestion** (offline, `scripts/ingest.py`) and **inference** (runtime, `app.py`/`main.py`). Detailed docs: `architecture.md` (retrieval mechanics), `KB_DESIGN.md` (KB v2 tiering), `QA_REPORT.md` (known failure modes).

- `config.py` — the single deployer touchpoint. `AgentConfig` dataclass + env-var overrides drive everything: LLM provider/model, embedding provider, retrieval knobs (`rag_k`, rerank threshold, canonical rescue cutoff), PII tiers, system prompt. Change behavior here, not in implementation files.
- `rag/pipeline.py` — `RetrievalPipeline`, the **single implementation of retrieval**. The LangChain tool (`rag/retriever.py`), diagnostics, and eval harness all call it — never reimplement Chroma/BM25/merge logic elsewhere; that drift is exactly what this module was created to fix.
- `core/rag_chat.py` — **the answer path for every entrypoint**: single-pass RAG (retrieve → one LLM call), stateless, history passed in by the caller. It replaced a LangGraph ReAct agent that spent a whole LLM round-trip per turn deciding to call `rag_search` — which the prompt mandated anyway. `stream()` yields `("context", str)` → `("delta", str)*` → `("done", Answer)`; the context event is how the Streamlit QA panel shows retrieved chunks without a second retrieval, and must never be forwarded to a public caller.
- `core/factory.py` + `core/llms.py` — `build_llm()` is the one place provider selection happens. There is no agent, no tool registry, and no `config.tools` any more; **never put tool instructions in `system_prompt`** — with no tool bound the model answers users with "I'll execute the `rag_search` tool first".
- `core/utils.py` — `enforce_grounding(grounded, text)` is the deterministic backstop. Its digit check allows exactly the phone numbers and the two company figures (`1998`, `10,000`) the prompt itself supplies; anything else numeric in an ungrounded reply becomes the canned refusal.
- `core/tools/registry.py` — tools registered by name (`register_tool("rag", ...)`); `main.py:_setup_rag_tool` wires retrieval and is reused by `app.py`.
- `core/utils.py` — response post-processing chain used by both UIs: `build_user_query`/`wrap_user_query` → PII query guard → `clean_response_prefix` → `enforce_grounding_refusal` (deterministic refusal backstop when rag_search returns nothing) → PII response guard.
- `data_handlers/` — loader registry pattern; `enriched_loader.py` reads `data/enriched_knowledge_base.json` and carries `authority`/`answer_policy` metadata into Chroma.
- `rag/guardrails/pii_detector.py` — Presidio-based, three checkpoints (ingest/query/output).
- `warmup.py` — `build_shared_config()` (lru_cached, process-global) builds config + registers the RAG tool + warms the cross-encoder. Every entrypoint calls it; `python warmup.py` also boots Streamlit *after* warming so the first visitor doesn't wait ~40s.
- `api/` — FastAPI service for the website chat widget. `api/chat.py` runs **one shared RagChat** with per-session history held in-process (Streamlit's agent-per-session model doesn't scale), and bounds everything a public deployment must bound: session TTL + count (MemorySaver would otherwise grow until OOM), first-turn answer cache, concurrent-turn semaphore, per-turn timeout. `api/main.py` adds per-IP rate limiting (slowapi), CORS allowlist, request ids, and sanitized errors. Knobs live in `APIConfig` in `config.py`. Single-instance assumptions are marked there — multi-instance needs `RATE_LIMIT_STORAGE_URI=redis://…` plus sticky sessions or externalized memory.
- `app.py` — Streamlit, now an internal/QA UI. Retrieval is shared process-wide; the agent stays per-session because its MemorySaver holds conversation state.

## Knowledge base (v2)

Every record in `data/enriched_knowledge_base.json` carries `authority` (canonical / company / evidence / background — whose voice) and `answer_policy` (answer / careful / deflect / route_client_line — how far an answer may go). These flow loader → vector-store metadata → LLM context. Scope guards are *records*: correct refusals made retrievable. Canonical Q&A gets a widened retrieval rescue window. Preserve both fields when adding or editing KB content; see `KB_DESIGN.md` before restructuring.

Raw scraped sources live in `data/raw/`; scrapers in `scripts/scrapers/`; `scripts/build_kb_v2.py` assembles the enriched KB from them. **Hand-authored answers go in `data/canonical_qa_v2.json`, never straight into `enriched_knowledge_base.json`** (the build overwrites it): add the record with `variant_questions`, then `build_kb_v2.py` → `ingest.py --loader enriched --force`.

Canonical records are deliberately chunked like everything else, even though it splits their `Also asked:` block into a separate answer-less chunk. That chunk is the vector informal phrasings match on — keeping canonical records whole dropped eval accuracy 1.00 → 0.97, because all-MiniLM-L6-v2 truncates at 256 tokens.

---
> Source: [psychedeliac/CT-chatbot](https://github.com/psychedeliac/CT-chatbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
