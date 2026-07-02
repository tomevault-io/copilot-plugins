## docker-swarm-mcp

> Auto-generated from all feature plans. Last updated: 2025-10-07

# docker-swarm-mcp Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-10-07

## Active Technologies
- Python 3.12 + FastAPI, Uvicorn, docker-py SDK, Pydantic

## Project Structure
## Project Structure

## Commands

# Common tasks for active technologies
# Run from the project root

```
# Run tests
pytest

# Lint the codebase
ruff check .

# Optional: type checks (if mypy is installed)
mypy app || true
```

## Code Style
Python 3.12: Follow standard conventions

## Recent Changes
- Added Python 3.12 + FastAPI, Uvicorn, docker-py SDK, Pydantic
- Implemented MCP JSON-RPC 2.0 protocol with tool gating and schema validation
- Enhanced security with token-based authentication and scope validation
- Added comprehensive error handling and retry mechanisms

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Source: [KHAEntertainment/docker-swarm-mcp](https://github.com/KHAEntertainment/docker-swarm-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
