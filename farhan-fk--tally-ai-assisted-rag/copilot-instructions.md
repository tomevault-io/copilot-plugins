## tally-ai-assisted-rag

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

This project uses `uv` as its package manager (Python >= 3.13 required by `pyproject.toml`).

```bash
# Install dependencies
uv sync

# Run the app (from repo root) - starts uvicorn with --reload on port 8000
./run.sh

# Run the app manually
cd backend && uv run uvicorn app:app --reload --port 8000
```

The app serves both the API and the static frontend from the same FastAPI process:
- Web UI: `http://localhost:8000`
- Swagger docs: `http://localhost:8000/docs`

Environment: requires a `.env` file in the repo root with `ANTHROPIC_API_KEY=...` (see `.env.example`). Loaded via `python-dotenv` in `backend/config.py`.

### Tests

Tests live in `backend/tests/` (pytest + `unittest.mock`; no real API/DB calls — the Anthropic client and `VectorStore` are mocked). Not currently in `pyproject.toml` dependencies — install with `uv pip install pytest pytest-mock` (or `pip install` inside the venv) if not already present.

```bash
cd backend
uv run pytest tests/ -v                     # full suite
uv run pytest tests/test_ai_generator.py -v # one file
uv run pytest tests/ -k test_name -v        # one test by name
```

`backend/tests/conftest.py` inserts `backend/` onto `sys.path` so test modules import top-level modules (`from search_tools import ...`) the same way `app.py` does at runtime — no package `__init__.py` layout.

There is no configured linter/formatter/type-checker in this repo.

## Architecture

This is a RAG (Retrieval-Augmented Generation) chatbot that answers questions about course materials. Three layers: static frontend → FastAPI backend → tool-calling AI orchestration over a ChromaDB vector store.

### Request flow

`frontend/script.js` → `POST /api/query` (`backend/app.py`) → `RAGSystem.query()` (`backend/rag_system.py`) → `AIGenerator.generate_response()` (`backend/ai_generator.py`), which calls the Anthropic API with tool definitions and, when Claude requests a tool, executes it via `ToolManager`/`CourseSearchTool` (`backend/search_tools.py`) against `VectorStore` (`backend/vector_store.py`, ChromaDB-backed).

**`RAGSystem`** (`rag_system.py`) is the top-level orchestrator wiring together `DocumentProcessor`, `VectorStore`, `AIGenerator`, `SessionManager`, and `ToolManager`. It does *not* embed search logic itself — it delegates to the AI generator + tools and just handles session history and returning `(answer, sources)`.

**Sequential tool calling** (`ai_generator.py`): `AIGenerator.generate_response()` runs a round-counter loop, not a single request/response. Claude may call a tool for up to `MAX_TOOL_ROUNDS = 2` rounds, each a separate `messages.create` API call, with full conversation context (assistant tool-use blocks + user tool-result blocks) preserved between rounds. Tools are offered on rounds 1–2; if round 2 also requests a tool, a final round-3 call is made *without* `tools`, forcing Claude to synthesize an answer from everything gathered rather than search again. A failed tool call (exception, not a "no results" string) short-circuits the loop immediately with a canned graceful message — no further API call is made.

**Source citations**: `CourseSearchTool._format_results()` (`search_tools.py`) tracks sources as it formats results, as `{"text": ..., "link": ...}` dicts (link resolved via `VectorStore.get_lesson_link()`/`get_course_link()`). Because a single query can now trigger multiple tool rounds, `last_sources` **accumulates** across calls within one query (`.extend()`, not overwrite) — `RAGSystem.query()` reads them via `ToolManager.get_last_sources()` after the full answer is generated, then calls `reset_sources()` to clear them for the next query. The frontend (`frontend/script.js`) renders each source as a clickable `<a target="_blank">` when `link` is present, falling back to plain text otherwise.

**Vector store** (`vector_store.py`) keeps two ChromaDB collections:
- `course_catalog`: one entry per course, ID = course title, metadata includes a JSON-serialized `lessons_json` (lesson number/title/link) — queried via semantic search to *resolve* a fuzzy `course_name` argument (e.g. "MCP") to an exact stored title before filtering.
- `course_content`: the actual chunked lesson text, metadata `{course_title, lesson_number, chunk_index}` — queried with an optional `$and` filter on course_title/lesson_number built from the resolved course name.

**Document ingestion** (`document_processor.py`): expects plain-text course files (see `docs/*.txt`) in a fixed format — first 3 lines are `Course Title:` / `Course Link:` / `Course Instructor:`, followed by `Lesson N: <title>` markers (optionally followed by a `Lesson Link:` line) and lesson body text. Each lesson's text is sentence-chunked (`chunk_text`, config-driven `CHUNK_SIZE`/`CHUNK_OVERLAP`) into `CourseChunk`s, with the first chunk of each lesson prefixed with lesson context for better retrieval. `RAGSystem.add_course_folder()` (invoked from `app.py`'s FastAPI startup event) loads `../docs` on server start and skips any course title already present in the vector store, so re-adding the same files is a no-op.

**Config** (`config.py`): a single dataclass instance (`config`) holding `ANTHROPIC_MODEL`, `EMBEDDING_MODEL` (`all-MiniLM-L6-v2` via `sentence-transformers`), chunk size/overlap, `MAX_RESULTS`, `MAX_HISTORY`, and `CHROMA_PATH`. Read once at import time; there's no runtime reload.

**Session/history**: `SessionManager` (`session_manager.py`) keeps an in-memory (non-persistent) dict of `session_id -> [Message]`, trimmed to `MAX_HISTORY * 2` entries. History is formatted into a plain string and appended to the system prompt on each `AIGenerator` call — it is not passed as multi-turn `messages`.

### Models (`backend/models.py`)

`Course` (title = unique ID, `course_link`, `instructor`, `lessons: List[Lesson]`) / `Lesson` (`lesson_number`, `title`, `lesson_link`) / `CourseChunk` (the unit actually stored in `course_content`, references `course_title`/`lesson_number` by value, not by object reference — the two ChromaDB collections are only joined at query time via those fields).

---
> Source: [farhan-fk/tally_ai_assisted_rag](https://github.com/farhan-fk/tally_ai_assisted_rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
