## claude-cognitive

> **Project:** Task management REST API

# Task API - Simple REST API Example

**Project:** Task management REST API
**Status:** Example/Demo
**Primary Goal:** Demonstrate claude-cognitive with a small, realistic project

---

## Quick Reference

| What | Where | Command |
|------|-------|---------|
| Start | `src/main.py` | `python src/main.py` |
| Test | `tests/` | `pytest` |
| API Docs | Browser | `http://localhost:8000/docs` |

---

## Architecture Overview

```
Client → FastAPI Server → SQLite Database
              ↓
         Task Manager
```

**Core Components:**
- **API Server** (`src/main.py`): FastAPI application
- **Task Manager** (`src/tasks.py`): Business logic
- **Database** (`src/database.py`): SQLite persistence

---

## Getting Started (for Claude)

This is a **minimal example** showing how to document a small project for claude-cognitive.

**Context structure:**
- `systems/local-dev.md` - Local development environment
- `modules/api.md` - API layer documentation
- `modules/tasks.md` - Task manager logic

The context router will:
- Keep recently mentioned files HOT
- Keep related files WARM
- Evict unmentioned files

---

## Development Workflow

**Daily:**
```bash
# Start dev server
python src/main.py

# Run tests
pytest

# Check API
curl http://localhost:8000/health
```

---

## Common Operations

**Create task:**
```bash
curl -X POST http://localhost:8000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Example task","description":"Do something"}'
```

**List tasks:**
```bash
curl http://localhost:8000/tasks
```

---

## Critical Files

| File | Purpose | Key Lines |
|------|---------|-----------|
| `src/main.py` | FastAPI app entry point | 15-30 (routes) |
| `src/tasks.py` | Task business logic | 10-25 (CRUD operations) |
| `src/database.py` | SQLite setup | 5-15 (connection) |

---

## Environment Variables

```bash
# Optional
export DATABASE_PATH=tasks.db  # Default: tasks.db
export API_PORT=8000           # Default: 8000
```

---

## Recent Changes

**2024-12-30:**
- Initial example setup
- Basic CRUD operations
- SQLite persistence

---

## For New Developers

This example demonstrates:
1. Minimal but complete project structure
2. How to organize documentation for claude-cognitive
3. Balance between too little and too much context

**Customize for your project:**
- Replace with your actual architecture
- Add more systems/modules as needed
- Keep docs updated as code changes

---

**Last Updated:** 2024-12-30
**Maintained By:** MirrorEthic LLC (Example)

---
> Source: [GMaN1911/claude-cognitive](https://github.com/GMaN1911/claude-cognitive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
