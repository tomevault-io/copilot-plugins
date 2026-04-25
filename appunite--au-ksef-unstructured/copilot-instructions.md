## au-ksef-unstructured

> Python microservice: PDF invoice → structured JSON extraction.

# CLAUDE.md

## Project Overview

Python microservice: PDF invoice → structured JSON extraction.
Pipeline: PDF → PyMuPDF (text) → Anthropic Claude (structured output) → JSON.

## Tech Stack

- **Framework:** FastAPI
- **PDF parsing:** PyMuPDF
- **LLM:** Anthropic Claude via `anthropic` SDK (Structured Output with `output_config`)
- **Config:** pydantic-settings
- **Package manager:** uv
- **Python:** 3.11.7

## Commands

- `make install` — install dependencies
- `make dev` — run dev server with hot reload
- `make test` — run tests
- `make lint` — ruff check
- `make format` — ruff format
- `make typecheck` — mypy

## Project Structure

- `src/app/main.py` — FastAPI app creation & exception handlers
- `src/app/config.py` — Settings via pydantic-settings
- `src/app/dependencies.py` — Auth dependency (bearer token)
- `src/app/routes/` — API route handlers
- `src/app/schemas/` — Pydantic request/response models
- `src/app/services/` — Business logic (PDFParser, LLMExtractor)
- `src/app/prompts/` — Prompt templates
- `tests/` — Unit and integration tests

## Coding Standards

- Follow SOLID principles — single responsibility per class/module
- DRY — extract shared logic, don't repeat yourself
- TDD — write tests first when implementing new features
- Type hints on all function signatures
- Use `ruff` for linting and formatting
- Keep functions small and focused
- Prefer dependency injection over global state
- Error handling: raise HTTPException with appropriate status codes at the route level, let services raise domain-specific exceptions

## Conventions

- Import order: stdlib → third-party → local (enforced by ruff)
- Use `async def` for route handlers
- Services are synchronous classes (PDF parsing and LLM calls are blocking)
- Settings loaded once via `get_settings()` with `@lru_cache`
- Auth via FastAPI `Depends(verify_token)` on protected endpoints

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/appunite) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
