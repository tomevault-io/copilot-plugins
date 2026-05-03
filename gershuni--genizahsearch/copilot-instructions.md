## genizahsearch

> This is a Hebrew manuscript search platform using NiceGUI (web) and PyQt6 (desktop).

# GenizahSearch - AI Assistant Rules

## Project Context
This is a Hebrew manuscript search platform using NiceGUI (web) and PyQt6 (desktop).
Backend is Supabase (PostgreSQL cloud). FastAPI backend was REMOVED in January 2026.

## Documentation Requirements

IMPORTANT: Keep documentation updated when making code changes!

### After making changes, check:
1. Does CLAUDE.md need updating? (architecture, env vars)
2. Does docs/guides/SUPABASE_GUIDE.md need updating? (schema changes)
3. Does docs/guides/DEPLOYMENT_TECHNICAL.md need updating? (server config)

### Before committing, run:
```bash
python scripts/check_docs.py
```

### Outdated terms to AVOID in documentation:
- FastAPI, backend server (removed)
- genizah-backend service (removed)
- DATABASE_URL (use SUPABASE_URL)
- port 8000 (removed)

## Code Style
- Python 3.11+
- Type hints encouraged
- Hebrew comments acceptable
- NiceGUI for web UI
- PyQt6 for desktop UI

## Key Files
- web/supabase_client.py - Database operations
- genizah_core.py - Search engine
- web/pages/ - Web app pages
- CLAUDE.md - AI context (read this!)

---
> Source: [gershuni/GenizahSearch](https://github.com/gershuni/GenizahSearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
