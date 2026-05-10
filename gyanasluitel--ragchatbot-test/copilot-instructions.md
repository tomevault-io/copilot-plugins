## ragchatbot-test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
uv sync

# Run the application
./run.sh
# or
cd backend && uv run uvicorn app:app --reload --port 8000
```

App runs at `http://localhost:8000`. API docs at `http://localhost:8000/docs`.

Requires a `.env` file in the project root:
```
ANTHROPIC_API_KEY=your-key-here
```

## Architecture

Full-stack RAG chatbot: vanilla JS frontend + FastAPI backend + ChromaDB vector store + Claude API.

**Request flow:**
1. Frontend (`frontend/script.js`) POSTs `{ query, session_id }` to `POST /api/query`
2. `backend/app.py` routes to `RAGSystem.query()`
3. `RAGSystem` fetches session history, calls `AIGenerator.generate_response()` with the `search_course_content` tool available
4. Claude decides whether to invoke the tool; if so, `ToolManager` executes `CourseSearchTool` → `VectorStore.search()`
5. `VectorStore` fuzzy-resolves course names via the `course_catalog` ChromaDB collection, then queries `course_content` chunks
6. Tool results are fed back to Claude for a final answer (two-call pattern)
7. Sources and answer returned to the frontend; session history updated in `SessionManager`

**Key components:**
- `backend/rag_system.py` — main orchestrator; only entry point for queries and document ingestion
- `backend/ai_generator.py` — wraps Anthropic SDK; handles the tool-use two-call loop
- `backend/vector_store.py` — ChromaDB with two collections: `course_catalog` (titles/metadata) and `course_content` (chunked text); course name resolution uses vector similarity on the catalog
- `backend/search_tools.py` — `Tool` ABC, `CourseSearchTool`, and `ToolManager`; add new tools by implementing `Tool` and registering with `ToolManager`
- `backend/document_processor.py` — parses `.txt`/`.pdf`/`.docx` files; expects format: `Course Title:`, `Course Link:`, `Course Instructor:` in first 3 lines, then `Lesson N: <title>` / `Lesson Link:` markers
- `backend/session_manager.py` — in-memory session history; capped at `MAX_HISTORY=2` exchanges

**Config** (`backend/config.py`): all tunables in one `Config` dataclass — model, chunk size/overlap, max results, history length, ChromaDB path.

**Document format** (see `docs/` for examples):
```
Course Title: <title>
Course Link: <url>
Course Instructor: <name>

Lesson 1: <title>
Lesson Link: <url>
<lesson content...>
```

---
> Source: [gyanasluitel/ragchatbot-test](https://github.com/gyanasluitel/ragchatbot-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
