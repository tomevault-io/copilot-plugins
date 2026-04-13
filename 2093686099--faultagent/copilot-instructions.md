## faultagent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

### Backend
```bash
conda activate faultagent          # Python 3.12 environment
pip install -r requirements.txt    # Install dependencies
python app.py                      # Dev server on :8000
# Production:
gunicorn -w 4 -k uvicorn.workers.UvicornWorker app:app --bind 0.0.0.0:8000
```

### Frontend
```bash
cd agent_fronted
npm install
npm run dev                        # Dev server on :9005
npm run build                      # Production build → dist/
```

### Knowledge Base
```bash
python rebuild_kb.py               # Rebuild FAISS index from pdfs/
python -c "from knowledge_base import init_knowledge_base; init_knowledge_base()"
```

### Tests
```bash
pytest                             # Run all tests (76 tests)
pytest tests/test_smoke.py         # Run specific test file
pytest -x                          # Stop on first failure
```

## Architecture

An AI agent system for industrial equipment fault diagnosis. A single LangGraph ReAct agent processes user messages through a middleware pipeline, invokes tools, and streams responses via SSE.

**Data flow:** Frontend (Vue 3) → SSE/REST → FastAPI (`app.py`) → LangGraph Agent → Tools → MySQL/FAISS/External APIs

**Module structure:**

| Module | Purpose |
|--------|---------|
| `app.py` (256 lines) | Entry point: FastAPI routes, agent creation, lifespan, static files, CORS |
| `streaming.py` | SSE token-level streaming (`token_stream_events`) |
| `middleware.py` | `build_middleware()` factory — assembles TodoList + dynamic prompt + Summarization |
| `config.py` | Centralized constants (chunk sizes, recursion limit, API URLs), env-var overridable |
| `utils.py` | Generic utilities: `sanitize_for_json`, `safe_json_dumps`, `parse_todos_from_tool_output` |
| `knowledge_base.py` | FAISS vector store creation/loading with Ollama embeddings |
| `prompts/system_prompt.py` | Main system prompt with 6-step diagnostic workflow |
| `prompts/dynamic_prompt.py` | `Context` dataclass + `identity_aware_prompt` for role-based injection |
| `tools/data_tools.py` | `extract_data`, `fig_inter` (share state via `globals()`) |
| `tools/sql_tools.py` | `sql_inter`, `get_sqltools()` with lazy singleton DB init |
| `tools/kb_tools.py` | `query_knowledge_base` |
| `tools/report_tools.py` | `save_report`, `save_html_report` |
| `tools/utility_tools.py` | `get_time`, `search_tool` |
| `tools/subagent/` | Fault explanation sub-agent (creates fresh LangGraph agent per invocation) |

**Databases:**
- MySQL — Sensor/business data (queried by tools at runtime)
- PostgreSQL — LangGraph conversation state persistence (`AsyncPostgresSaver`)

**SSE event types:** `start`, `token`, `tool_start`, `tool_end`, `complete`, `server_error`

**Tests:** `tests/` directory with `conftest.py` (FakeToolCallingModel for mocking), `pytest-asyncio` for async tests

## Key Patterns

- **Tool definition**: Pydantic `BaseModel` schema + `@tool(args_schema=...)` decorator. Docstrings are in Chinese and serve as the LLM's tool description.
- **`@dynamic_prompt`**: Injects role-based system prompts at runtime based on `Context.user_identity` (游客/管理员).
- **`globals()` sharing**: `extract_data` and `fig_inter` share state via `globals()` in `data_tools.py` — `extract_data` stores DataFrames that `fig_inter` later reads. This is intentional.
- **Lazy DB initialization**: SQL tools use a `_db = None` / `_get_db()` singleton pattern; `get_sqltools()` defers loading until app lifespan.
- **Sub-agent as tool**: `fault_explanation_tool` creates a fresh sub-agent per invocation with its own tools and prompt.
- **Frontend path alias**: `@` → `agent_fronted/src/` in imports.

## Language & Style

- **All comments, docstrings, user-facing strings, and system prompts are in Chinese (Simplified)**
- Status prefixes: `✅` success, `❌` error, `⚠️` warning, `🚀` startup
- Python: `snake_case` files/functions, 4-space indent, double quotes preferred
- Vue: `PascalCase.vue` components, `camelCase.ts` composables prefixed with `use`
- Environment variables loaded via `python-dotenv` from `.env` at project root

## Configuration

Non-secret config is centralized in `config.py` with env-var overrides. Secret config lives in `.env` (never read its contents — contains secrets):
- **`config.py`**: `OLLAMA_BASE_URL`, `EMBEDDING_MODEL`, `FAISS_PATH`, `KB_CHUNK_SIZE`, `KB_CHUNK_OVERLAP`, `KB_BATCH_SIZE`, `MAX_TOKENS_BEFORE_SUMMARY`, `MESSAGES_TO_KEEP`, `RECURSION_LIMIT`, `DCMA_DB_NAME`, `FAULT_API_URL`
- **`.env` secrets**: MySQL (`HOST`, `USER`, `MYSQL_PW`, `DB_NAME`, `PORT`), PostgreSQL (`POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`), LLM (`MODEL_NAME`, `OPENAI_BASE_URL`, `OPENAI_API_KEY`)

## Constraints

- **Tech Stack**: LangChain 1.0.3 + LangGraph 1.0.5 + FastAPI 0.121.0 — do not upgrade
- **API Contract**: Do not change existing HTTP endpoints — frontend depends on them unchanged
- **No new heavy dependencies** — keep the project lightweight

## Project Status

v1.0 modular refactoring milestone is complete (5 phases, all verified). The original monolith has been split into the module structure above. Planning artifacts are in `.planning/` (STATE.md, PROJECT.md, MILESTONES.md).

## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2093686099)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/2093686099)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
