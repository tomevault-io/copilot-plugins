## obsidianrag

> ObsidianRAG is a RAG (Retrieval-Augmented Generation) system for querying Obsidian notes. Version 3 is structured as:

# ObsidianRAG v3 - Copilot Instructions

## Project Overview

ObsidianRAG is a RAG (Retrieval-Augmented Generation) system for querying Obsidian notes. Version 3 is structured as:
- **Backend**: Python package (`obsidianrag`) distributed via PyPI with CLI
- **Frontend**: TypeScript Obsidian plugin with native chat UI

## Repository Structure

```
ObsidianRAG/
├── backend/                    # Python backend (PyPI package)
│   ├── obsidianrag/           # Main package
│   │   ├── __init__.py        # Package exports, version
│   │   ├── __main__.py        # python -m obsidianrag entry
│   │   ├── config.py          # Pydantic Settings
│   │   ├── api/server.py      # FastAPI server + SSE streaming
│   │   ├── cli/main.py        # Typer CLI
│   │   └── core/              # Business logic
│   │       ├── db_service.py      # ChromaDB management
│   │       ├── qa_agent.py        # LangGraph RAG agent
│   │       ├── qa_service.py      # Hybrid retriever
│   │       └── metadata_tracker.py # Incremental indexing
│   └── tests/                 # pytest tests (77 tests)
│
├── plugin/                    # Obsidian plugin (TypeScript)
│   ├── src/main.ts           # All-in-one plugin (1800+ lines)
│   ├── tests/                # Jest tests (28 tests)
│   └── styles.css            # UI styles
│
├── docs/                     # User documentation
│   ├── CONTRIBUTING.md
│   ├── TESTING_GUIDE.md
│   └── TROUBLESHOOTING.md
│
└── .github/instructions/     # Development guidelines (for AI agents)
```

## Quick Reference

### Backend Commands (from `backend/`)
```bash
uv run obsidianrag serve --vault /path --model gemma3   # Start server
uv run obsidianrag serve --vault /path --no-reranker    # Without reranker
uv run pytest tests/ -v                                  # Run tests
uv run ruff check obsidianrag/ tests/ --fix             # Lint + fix
```

### Plugin Commands (from `plugin/`)
```bash
pnpm install          # Install deps
pnpm run dev          # Watch mode build
pnpm run build        # Production build
pnpm test             # Run Jest tests
```

## Key Technologies
- **Python**: 3.11+ with uv, FastAPI, LangGraph, ChromaDB
- **TypeScript**: Obsidian API, esbuild, Jest
- **LLM**: Ollama (local) - models fetched via `/api/tags`
- **Streaming**: SSE (Server-Sent Events) for real-time responses

## Architecture Patterns

### Data Flow
```
Question → Hybrid Search (BM25+Vector) → Rerank → GraphRAG → LLM → Stream Answer
```

### Plugin ↔ Backend Communication
- Plugin spawns backend via `child_process.spawn()`
- Communication via HTTP (`localhost:8000`)
- Streaming via SSE (`/ask/stream` endpoint)
- Models fetched from Ollama API (`localhost:11434/api/tags`)

### CLI Args from Plugin
Plugin passes settings to backend via CLI:
```typescript
spawn(pythonPath, ["serve", "--vault", path, "--model", model, "--reranker"]);
```

## Development Guidelines

📚 **See `.github/instructions/` for detailed patterns:**

| File | Topics |
|------|--------|
| `python-backend.md` | Python code style, Pydantic, FastAPI patterns |
| `testing.md` | pytest fixtures, mocking ChromaDB/Ollama |
| `obsidian-plugin.md` | TypeScript, Obsidian API, UI components |
| `rag-patterns.md` | LangGraph nodes, hybrid search, reranking |
| `streaming-patterns.md` | **SSE streaming, Ollama API, dynamic UI** |
| `ci-cd.md` | GitHub Actions, PyPI publishing |

## Critical Patterns

### Ollama Model Detection
```typescript
// Fetch available models from Ollama
const response = await fetch("http://localhost:11434/api/tags");
const { models } = await response.json();
```

### Kill Server by Port (cross-platform)
```typescript
// macOS/Linux
exec(`lsof -ti:${port} | xargs kill -9`);
// Windows
exec(`for /f "tokens=5" %a in ('netstat -aon | find ":${port}"') do taskkill /F /PID %a`);
```

### Auto-refresh Settings Status
```typescript
setInterval(() => this.updateStatusDisplay(), 3000);
```

## Project Status

- ✅ Phase 1-4: Backend + Plugin complete
- ✅ Phase 5: Testing (macOS ✅, Windows/Linux pending)
- ✅ Phase 6: Documentation complete
- ⏳ Phase 7: Community Plugins submission

See `V3_MIGRATION_PLAN.md` for full details.

---
> Source: [Vasallo94/ObsidianRAG](https://github.com/Vasallo94/ObsidianRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
