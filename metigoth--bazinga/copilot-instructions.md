## bazinga

> This file documents the contributions, state, and guidelines for the Gemini AI assistant (Tuesday, December 30, 2025).

# Gemini - AI Assistant Documentation

This file documents the contributions, state, and guidelines for the Gemini AI assistant (Tuesday, December 30, 2025).

## Recent Contributions (2025-12-30)

### 1. Connectivity & Initialization Fixes
- **Resolved 503 Service Unavailable:** Fixed backend crash caused by missing `SECRET_KEY`.
- **Resolved 401 Unauthorized:** Set `ENABLE_AUTH=false` by default in new `.env` to allow frontend connectivity without prior user setup.
- **Native Windows Support:** Moved `.env` to project root and updated `backend/app/core/config.py` to support loading from `../.env` when running natively.

### 2. Settings API Improvements
- **Append Support:** Updated `write_env_file` in `backend/app/api/settings.py` to append new configuration keys instead of ignoring them.
- **Redirect Fix:** Added trailing slash to `/api/settings/` in frontend to prevent 307 redirects.
- **Docker Override:** Created `docker-compose.override.yml` to map local `.env` for Docker environments.

### 3. Search Precision (Regex Engine)
- **Spinoff Exclusion Logic:** Implemented a negative lookahead system in `verification_service.py` to prevent short series titles from matching spinoffs (e.g., "Grimm" vs "Grimm Fairy Tales").
- **Exclusion Map:** Added specific entries for `grimm`, `star trek`, `batman`, etc.

## Core Mandates & Conventions

### Coding Style
- **Backend:** FastAPI, SQLAlchemy, Pydantic. Use `get_logger(__name__)` for logging.
- **Frontend:** React with TypeScript, Vite. Services located in `frontend/src/services`.
- **Formatting:** Adhere to existing spacing and naming conventions (camelCase in TS, snake_case in Python).

### Key Files
- **Configuration:** `.env` (Project Root). Handled by `backend/app/core/config.py`.
- **Models:** `backend/app/models/`
- **Services:** `backend/app/services/`
- **Documentation:** `CLAUDE.md` (Technical Architecture), `README.md` (Setup), `todo.md` (Tasks).

## Development State

- **Environment:** Running natively on Windows.
- **Database:** `backend/bazinga.db` (SQLite).
- **Authentication:** Currently disabled via `ENABLE_AUTH=false` for development ease.

## Remaining Tasks (Gemini Priority)
- [ ] Implement user creation/login UI to re-enable `ENABLE_AUTH`.
- [ ] Expand spinoff exclusion map as new edge cases are found.
- [ ] Address Path Traversal warning in `webhooks.py` (noted in `todo.md`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Metigoth)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Metigoth)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
