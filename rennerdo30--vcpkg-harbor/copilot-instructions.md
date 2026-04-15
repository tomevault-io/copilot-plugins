## vcpkg-harbor

> Binary cache server for vcpkg with plugin-based storage backends. A Python FastAPI application that implements the vcpkg binary caching protocol.

# vcpkg-harbor - Claude Instructions

## Project Overview

Binary cache server for vcpkg with plugin-based storage backends. A Python FastAPI application that implements the vcpkg binary caching protocol.

## Technology Stack

- **Framework**: FastAPI with Python 3.10+
- **Storage**: Plugin-based (MinIO, Filesystem, S3, Azure, GCS)
- **Database**: None (stateless, storage backend handles persistence)
- **Auth**: Optional (basic auth, API key, or anonymous)

## Key Commands

```bash
# Development
uvicorn vcpkg_harbor.app:create_app --reload

# Testing
pytest tests/ -v
pytest --cov=vcpkg_harbor tests/

# Docker
docker-compose up

# Linting
ruff check .
black .
mypy .
```

## Project Structure

```
src/vcpkg_harbor/
├── app.py                # FastAPI application factory
├── core/                 # Core infrastructure
├── api/                  # REST API endpoints
├── storage/              # Storage backends
├── services/             # Business logic
├── auth/                 # Authentication
└── dashboard/            # Web UI
```

## Important Rules

1. **Keep SPECIFICATION.md up to date** with any changes
2. **Run tests before committing** - `pytest tests/`
3. **Follow existing patterns** in the codebase
4. **Use type hints** throughout

## Documentation

- **Always use Mermaid** for diagrams in documentation
- Mermaid is supported in the Starlight docs site
- Avoid external diagram tools or image files when possible

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rennerdo30)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rennerdo30)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
