## ragchatbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**IMPORTANT:** This project uses `uv` as the package manager. **Never use `pip` directly.** All Python commands must be run with `uv run` prefix.

### Environment Setup
```bash
# Install dependencies
uv sync

# Install dev dependencies (for linting, formatting, type checking)
uv sync --group dev

# Environment variables required
# Create .env file in root with:
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

### Running the Application
```bash
# Quick start using shell script
chmod +x run.sh
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000

# Access points:
# - Web Interface: http://localhost:8000
# - API Documentation: http://localhost:8000/docs
```

### Testing
```bash
# Run all tests
uv run pytest

# Run specific test file
uv run pytest backend/tests/test_rag_system.py

# Run tests by marker
uv run pytest -m unit          # Unit tests only
uv run pytest -m integration   # Integration tests only
uv run pytest -m api          # API tests only
uv run pytest -m "not slow"   # Skip slow tests

# Run single test
uv run pytest backend/tests/test_rag_system.py::test_query_with_session
```

### Code Quality
```bash
# Auto-fix formatting and imports (modifies files)
./scripts/format.sh

# Check code quality without modifying (for CI/pre-commit)
./scripts/lint.sh
```

### Python Execution
Always use `uv run` prefix for Python commands:
```bash
uv run python script.py
uv run pytest
```

### Adding Dependencies
```bash
# Production dependency (DO NOT use pip install)
uv add package_name

# Development dependency (DO NOT use pip install)
uv add --group dev package_name
```

## Architecture Overview

This is a Retrieval-Augmented Generation (RAG) system for course materials. The key architectural pattern is **tool-based search**: Claude decides when to search vs. use general knowledge.

### Request Flow (User Query → Response)

1. **Frontend** (`frontend/script.js`) → POST `/api/query` with `{query, session_id}`
2. **FastAPI** (`backend/app.py`) → Create/retrieve session → Call `rag_system.query()`
3. **RAGSystem** (`backend/rag_system.py`) → Retrieve conversation history → Pass tools to AI
4. **AIGenerator** (`backend/ai_generator.py`) → Claude API with **up to 2 rounds of tool calls**
5. **Tool Execution** (`backend/search_tools.py`):
   - Claude calls `search_course_content` with query, optional course_name, lesson_number
   - Tool tracks sources/links for UI attribution
6. **VectorStore** (`backend/vector_store.py`):
   - Resolve course name via semantic search in `course_catalog` collection
   - Build filter (course_title, lesson_number)
   - Query `course_content` collection with embeddings
7. **Response** → Tool formats results → Claude synthesizes answer → RAG collects sources → FastAPI JSON → Frontend renders

### Dual Collection Strategy (ChromaDB)

**Why Two Collections?**
The system uses separate collections for metadata and content to enable intelligent course resolution:

1. **`course_catalog`**: Stores course metadata for semantic name matching
   - Documents: Course titles (searchable text)
   - IDs: Course title (unique identifier)
   - Metadata: `title`, `instructor`, `course_link`, `lesson_count`, `lessons_json`
   - Purpose: Fuzzy matching ("MCP" → "Introduction to MCP")

2. **`course_content`**: Stores chunked lesson content for retrieval
   - Documents: Text chunks (800 chars, 100 char overlap)
   - IDs: `{course_title}_{chunk_index}`
   - Metadata: `course_title`, `lesson_number`, `chunk_index`
   - Purpose: Semantic search within filtered course/lesson scope

**Search Flow Example:**
```
User: "What is MCP in lesson 3?"
  ↓
course_catalog.query("MCP") → course_title = "Introduction to MCP"
  ↓
course_content.query("What is MCP", where={
  "course_title": "Introduction to MCP",
  "lesson_number": 3
})
```

### Session Management

Sessions are **in-memory only** and cleared on server restart:
- Auto-created on first query if no `session_id` provided
- History limited to last 2 message pairs (configurable via `MAX_HISTORY` in config)
- Used to provide conversation context to Claude
- Format: `"User: {query}\nAssistant: {response}"`

### Document Processing Pipeline

When documents are added to `docs/` and server starts:

1. **Parse Document** (`document_processor.py`):
   ```
   Expected format:
   Course Title: [title]
   Course Link: [url]
   Course Instructor: [name]

   Lesson 1: [title]
   Lesson Link: [url]
   [content...]
   ```

2. **Chunk Text**: Sentence-based splitting (800 chars, 100 overlap)
   - First chunk of lesson: Prefixed with `"Lesson {N} content:"`
   - Preserves lesson context in chunks

3. **Store in ChromaDB**:
   - Course metadata → `course_catalog`
   - Content chunks → `course_content`

4. **Deduplication**: Existing course titles (by exact match) are skipped on reload

### Tool Calling Pattern

**Multi-Round Tool Use:**
- Claude can make **up to 2 sequential rounds** of tool calls
- Use cases:
  - Round 1: Search broad topic → Round 2: Refine with specific lesson
  - Round 1: Get course outline → Round 2: Search specific lesson content
- After 2 rounds or when Claude stops using tools, final response is generated

**Available Tools:**
1. **`search_course_content`** (CourseSearchTool):
   - Parameters: `query` (required), `course_name` (optional), `lesson_number` (optional)
   - Returns: Formatted chunks with `[Course - Lesson N]` headers
   - Side effect: Stores `last_sources` and `last_source_links` for UI

2. **`get_course_outline`** (CourseOutlineTool):
   - Parameters: `course_name` (required)
   - Returns: Course title, link, and complete lesson list

### Key Configuration Values

Located in `backend/config.py`:
- `CHUNK_SIZE`: 800 characters
- `CHUNK_OVERLAP`: 100 characters
- `MAX_RESULTS`: 5 search results per query
- `MAX_HISTORY`: 2 conversation pairs
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2 (SentenceTransformers)
- `CHROMA_PATH`: ./chroma_db

### Testing Architecture

Test files mirror backend structure:
- `conftest.py`: Shared fixtures (temp directories, mock data, test courses)
- Markers: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.api`, `@pytest.mark.slow`
- API tests use `TestClient` from FastAPI
- Vector store tests use temporary ChromaDB instances

## Important Patterns

### When Modifying Search Behavior
1. Update tool definition in `search_tools.py` (parameters, description)
2. Update `AIGenerator.SYSTEM_PROMPT` if tool usage guidelines change
3. Tool results format affects Claude's synthesis - keep headers consistent

### When Adding New Course Documents
- Place in `docs/` directory
- Follow expected format (see Document Processing Pipeline)
- Server auto-loads on startup
- Duplicates (same course title) are skipped

### When Changing Embedding Model
- Update `EMBEDDING_MODEL` in config
- Clear existing ChromaDB data: Delete `backend/chroma_db/`
- Restart server to rebuild embeddings

### Source Attribution Flow
1. `CourseSearchTool.execute()` stores sources in `self.last_sources` and `self.last_source_links`
2. `ToolManager.get_last_sources()` retrieves them after AI generation
3. `RAGSystem.query()` collects and returns to API
4. Frontend displays as clickable links

### Frontend Architecture
- **Vanilla JS** - no build step required
- `marked.js` (CDN) for markdown rendering
- Theme persistence in localStorage
- Session management: `currentSessionId` tracked globally
- Cache-busting: Version query params on static files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/georgexzeng)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/georgexzeng)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
