## dynamous-ai-coach

> RAG AI Agent that acts as an AI coach that has access to YouTube transcripts through various RAG tools.

# AI Agent Development Instructions

## Project Overview

RAG AI Agent that acts as an AI coach that has access to YouTube transcripts through various RAG tools.
RAG pipeline that pulls YouTube transcripts from a specific YouTube channel and puts them in a vector database to be queried by the agent.

## Core Principles

1. **TYPE SAFETY IS NON-NEGOTIABLE**
   - All functions, methods, and variables MUST have type annotations
   - Strict mypy configuration is enforced
   - No `Any` types without explicit justification

2. **KISS** (Keep It Simple, Stupid)
   - Prefer simple, readable solutions over clever abstractions. Avoid overengineering.

3. **YAGNI** (You Aren't Gonna Need It)
   - Don't build features until they're actually needed

**Architecture:**
```
src/
├── agent/          # Core AI coach
├── tools/          # Independent slices (rag_tools, web_search, etc.)
├── rag_pipeline/   # YouTube video RAG pipeline
└── utils/          # Cross-cutting (config, logging, middleware)
```

Each tool is a vertical slice containing simple tool.py, schemas.py, service.py.

---

## Important reference guides:

- When building tools, reference the guide: `PRPs/ai_docs/tool_guide.md`

- When implementing logging, reference the guide: `PRPs/ai_docs/logging_guide.md`

- When implementing tests, reference the guide: `PRPs/ai_docs/testing_guide.md`

---

## Documentation Style

**Use Google-style docstrings** for all functions, classes, and modules:

```python
def process_request(user_id: str, query: str) -> dict[str, Any]:
    """Process a user request and return results.

    Args:
        user_id: Unique identifier for the user.
        query: The search query string.

    Returns:
        Dictionary containing results and metadata.

    Raises:
        ValueError: If query is empty or invalid.
        ProcessingError: If processing fails after retries.
    """
```

---

## Development Workflow

**Run server:** `uv run uvicorn src.main:app --host 0.0.0.0 --port 8030 --reload`

**Lint/check (must pass):** `uv run ruff check src/ && uv run mypy src/`

**Auto-fix:** `uv run ruff check --fix src/`

**Run tests:** `uv run pytest tests/ -v`

---

## Adding Features

1. Create vertical slice in `src/tools/<name>/`
2. Define Pydantic schemas in `schemas.py` (types first)
3. Implement with proper logging, docstrings, and type annotations
4. **Create corresponding test file** in `tests/tools/<name>/test_<module>.py`
5. Write unit tests for the component
6. Add integration tests if the feature interacts with other components
7. Verify linters pass: `uv run ruff check src/ && uv run mypy src/`
8. Ensure all tests pass: `uv run pytest tests/`

---

## AI Agent Notes

When debugging:
- Check `source` field for file/function location
- Use `correlation_id` to trace full request flow
- Look for `duration_ms` to identify bottlenecks
- Exception logs include full stack traces with local variables (dev mode)
- All context is in structured log fields—use them to understand and fix issues

---
> Source: [coleam00/dynamous-ai-coach](https://github.com/coleam00/dynamous-ai-coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
