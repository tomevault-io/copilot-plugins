## clarifi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Backend
source .venv/bin/activate
pip install -e ".[dev]"
uvicorn clarifi.main:app --reload --port 8000

# Frontend
cd frontend && npm install && npm run dev

# Tests
pytest tests/ -v                              # all tests
pytest tests/test_validators.py -v            # single file
pytest tests/test_tools.py::test_query_cashflow -v  # single test
pytest tests/test_agent_e2e.py -v -s          # e2e (needs GOOGLE_API_KEY)

# Lint
ruff check src/clarifi/ scripts/ --select F   # real errors only
ruff check src/clarifi/ --fix                 # auto-fix

# Seed test data
python -m scripts.seed_db

# Frontend build
cd frontend && npx next build
```

## Architecture

Clarifi is an AI financial assistant for Romanian services companies. It uses a **LangGraph ReAct agent** with **Gemini** as the LLM, backed by SQLAlchemy async (SQLite dev / Supabase PostgreSQL prod).

### Two Agent Modes

The same graph serves two structurally different modes, controlled by `mode` in `AgentState`:

- **`mode="interactive"`** (default): LLM selects skills via `select_skills_llm`, all 36 tools available including `ask_user`, natural Romanian conversation with company context.
- **`mode="background"`**: Forces `background_processing.md` skill (bypasses LLM routing), removes `ask_user` tool, adds background instructions to system prompt. Used by file watcher and upload auto-processing.

### Graph Flow (`agent/graph.py`)

```
START → skill_loader → memory_retriever → react_agent_node → memory_saver → END
```

- `skill_loader`: Background mode → forces background skill. Interactive → LLM picks from 17 `.md` skill files.
- `memory_retriever`: Fetches context from memU (if enabled). Scoped per `user_id`.
- `react_agent_node`: Creates a `create_react_agent` with Gemini + logged tools + system prompt (company context + skill instructions + memory).
- `memory_saver`: Fire-and-forget store to memU.

### Skills = `.md` Files (`skills/`)

Skills are markdown files loaded into the LLM's context. Each has: Keywords (for fallback matching), Tools (which tools to bind), Instructions (what the LLM should do), and tone examples. The LLM routing (`skills/loader.py:select_skills_llm`) picks 1-3 skills per turn using structured output.

### Tools = `@tool` Functions (`tools/`)

36 async Python functions decorated with LangChain's `@tool`. Each queries the DB via `get_async_session()` and returns a dict. Tool calls are auto-logged to `decision_log` via `agent/logging_middleware.py`.

### Background Systems (`main.py` lifespan)

Two `asyncio.create_task` background loops start with the server:
- **Scheduler** (`scheduler.py`): Polls `scheduled_tasks` every 60s, fires due tasks through the agent graph with `mode="background"`.
- **File Watcher** (`discovery/watcher.py`): Polls `inbox/` every 10s, triggers background agent for new files with 5-minute timeout.

### Data Model

Key entities: `Company`, `Contract` (with `ContractMilestone`, `ContractObligation`, `ContractPenalty`), `Invoice` (with `InvoiceLineItem`), `BankTransaction`, `PaymentInvoiceMatch`, `Project`, `Estimate`, `Document`. Supporting: `ScheduledTask`, `SchedulerRun`, `DecisionLog`, `AgentSession`, `UserProfile`, `VirtualFolder`, `FileEntry`, `WatchedFolder`, `IntegrationConfig`, `Alert`, `AuditLog`.

Mixins on `models/base.py`: `AuditMixin` (created/updated timestamps), `SoftDeleteMixin` (is_deleted flag), `SourceTraceableMixin` (source_document_id, extraction_confidence), `FreshnessMixin` (verified_at, freshness_status — on Invoice, Contract, BankTransaction).

### Virtual Folder Tree (`models/file_tree.py`)

Documents are organized in a virtual tree stored in DB (`VirtualFolder` + `FileEntry`), not filesystem directories. Each folder has trace fields (`trace_content`, `trace_summary`, `trace_findings`) where the agent writes analysis notes.

### Extraction Pipeline

`tools/extraction.py`: Romanian prompt with Pydantic validation (`ExtractedInvoice`, `ExtractedContract`, `ExtractedBankStatement`), few-shot examples, 16k token limit. Post-extraction: `tools/validators.py` normalizes DD.MM.YYYY dates, comma-decimal amounts, CUI format. `tools/documents.py:save_extracted_data` handles invoice, contract, and bank_statement entity types with `_safe_decimal` and `_to_date` helpers.

### Per-User Isolation

`user_id` extracted from Supabase JWT in `api/chat.py:_extract_user_id`. Flows through `AgentState` → memU (scoped retrieval/storage), LangGraph checkpointer (thread prefixed with `user_id:`), company context (`api/onboarding.py:get_company_context`). `VirtualFolder` and `ScheduledTask` have `user_id` columns.

## Key Conventions

- All prompts and skill files are in **Romanian** (the product targets Romanian companies). System prompt is in `agent/prompts.py`.
- The agent personality is casual ("tu" not "dumneavoastră"), direct, empathic. Skills use NU/DA examples to teach tone by contrast.
- `config.py` uses pydantic-settings with `.env` file. All settings have defaults that work for local SQLite development.
- DB session pattern: `async with get_async_session() as session:` — auto-commits on exit, auto-rollbacks on exception.
- Tests patch the session factory via `monkeypatch.setattr(session_module, "async_session_factory", factory)`.
- The graph singleton (`_compiled_graph`) uses `AsyncSqliteSaver(conn=await aiosqlite.connect("memu.db"))` for the checkpointer — not `from_conn_string` (which is a context manager).

---
> Source: [aboros98/clarifi](https://github.com/aboros98/clarifi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
