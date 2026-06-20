## medical-citation-chatbot

> @.claude/skills/medical-rag/SKILL.md

@.claude/skills/medical-rag/SKILL.md

# Project: Medical Chatbot (Generative RAG)

## Quickstart

```bash
# Backend
cd backend
source .venv/bin/activate
pip install -r requirements.txt
ollama pull qwen2.5:7b
cd .. && find backend -type d -name __pycache__ -exec rm -rf {} + && PYTHONPATH=. uvicorn backend.main:app --reload --port 8000

# Frontend
cd frontend
npm install
npm run dev
```

## Architecture

```
User → Next.js (port 3000) → FastAPI (port 8000) → Wikipedia / OpenFDA / Ollama
                                   ↑
                          SSE: token | citation | done | error | warning | info
```

### Pipeline Flow (No Classifier, No Hardcoded Prompt)

```
User query (any medical question — accepted unconditionally)
  ↓
Phase 1: Wikipedia — search raw query, get extracts
  ↓ (fallback: search extracted drug names if no articles found)
Phase 2: Heuristic drug name extraction (capitalized + 5-15 char lowercase, minus stop words)
  ↓
Phase 3: OpenFDA — concurrent drug label searches for extracted names
  ↓
Phase 4: Citation metadata — build citation list from Wiki + FDA results
  ↓
Phase 6: Build minimal prompt (context + "answer helpfully, cite [[CITATION:N]], include disclaimer")
  ↓
Phase 7: Stream — citation metadata → info/warning → qwen2.5:7b token stream
  ↓
Phase 8: Persist conversation to session store
  ↓
Citation post-processing: normalise markers → filter used citations → done event
```

## Key Files

| File | Purpose |
|------|---------|
| `backend/symptom_pipeline.py` | Self-contained pipeline: wiki → drug extract → OpenFDA → prompt → LLM → citations (all formatting inline, no prompts.py) |
| `backend/wiki_client.py` | MediaWiki API: raw query search + plain-text extracts (no suffix bias) |
| `backend/openfda_client.py` | OpenFDA drug/label: Rx + OTC field extraction, DailyMed links |
| `backend/rxnav_client.py` | RxNorm API: drug name → RxCUI (UNWIRED — not called by pipeline, kept for reference) |
| `backend/config.py` | Centralised config: all endpoints, timeouts, model name, prompt limits |
| `backend/retry.py` | Shared HTTP retry with Retry-After parsing, exponential backoff |
| `backend/session_store.py` | In-memory conversation history (6-turn window, 30-min TTL) |
| `backend/logging_setup.py` | Structured logging with request-ID via context var |
| `backend/main.py` | FastAPI server, POST /api/chat SSE dispatch with try/except guard |
| `frontend/hooks/useChatController.ts` | Single hook: reducer + stream + session store + debounced persistence |
| `frontend/hooks/useChatStream.ts` | SSE consumer (fetch + ReadableStream + CRLF-safe parsing) |
| `frontend/hooks/useChatReducer.ts` | 12-action useReducer for chat state |
| `frontend/hooks/useSessionStore.ts` | localStorage-backed sessions (initialized in useState, not useEffect) |
| `frontend/components/MessageBubble.tsx` | Renders text with [[CITATION:N]] → inline [Wikipedia ↗] / [FDA ↗] tags |
| `frontend/components/InlineCitation.tsx` | Clickable source-labeled inline citation badges |
| `frontend/components/CitationPill.tsx` | Rich citation pills below message (title, source, external link) |

## Conventions

- **Imports:** Backend uses absolute imports (`from backend.wiki_client import …`). Always run from project root with `PYTHONPATH=.`.
- **Error handling:** API clients return empty/fallback data on failure — never crash the caller. External calls always have timeouts. SSE stream always terminates with `done` or `error`.
- **Citation format:** `[[CITATION:N]]` markers. Backend normalises `[N]`, `(N)`, `[[CITATION N]]` → `[[CITATION:N]]`. Frontend renders as `[Wikipedia ↗]` (teal) / `[FDA ↗]` (amber) inline badges.
- **Concurrency:** OpenFDA drug searches run via `asyncio.gather()` in parallel. Wikipedia uses shared `AsyncClient` instance.
- **Streaming:** `_stream_ollama()` yields `(event_type, data_dict)` tuples — the caller serialises to SSE so full_text can be tracked without re-parsing.
- **Logging:** Structured with `[request_id]` column via context var. Every `logger.info/warning` call is traceable.
- **No hardcoded prompts:** `_build_prompt()` in `symptom_pipeline.py` is minimal — context sections + "answer helpfully, cite sources, include disclaimer". No system prompt, no role constraints.
- **Pycache:** Always run `find backend -type d -name __pycache__ -exec rm -rf {} +` after code changes before restarting uvicorn.
- **Python deps:** Always activate the venv (`source backend/.venv/bin/activate`) before running anything.
- **Deleted files:** `prompts.py`, `query_classifier.py`, `classifier_data.py`, `pubmed_client.py`, `semantic_scholar_client.py`, `rag_pipeline.py` — do not recreate any of these.

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->

---
> Source: [MHKaungPyae/Medical_Citation_Chatbot](https://github.com/MHKaungPyae/Medical_Citation_Chatbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
