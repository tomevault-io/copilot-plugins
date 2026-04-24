## rag-multi-tenant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Project Context

## Project Overview
Personal website template with an intelligent RAG-powered AI assistant. Backend built with FastAPI, frontend with Astro. The backend uses a **unified smart retriever system** that automatically discovers, indexes, and intelligently routes queries to relevant content without manual configuration. Features a comprehensive admin dashboard for monitoring and analytics.

## Key Commands

### Development
- `npm run dev` - Start Astro frontend dev server
- `npm run backend:dev` - Run backend with hot reload
- `npm run backend:dev:reindex` - Force reindex data
- `npm run admin:frontend` - Start admin dashboard
- `make lint-fast` - Quick format (Black + isort)
- `make test-unit` - Run unit tests only
- `pytest -m unit` - Fast unit tests

### Deployment
- `npm run build` - Build Astro frontend
- `npm run backend:build` - Build backend container
- `npm run admin:build` - Build admin frontend
- `npm run railway:deploy` - Deploy to Railway

### Code Quality (Run before commits)
- `make lint-fix` - Auto-format all code
- `make lint` - Full lint pipeline
- `pytest` - Run all tests with coverage

## API Endpoints

### Main Query API
- `POST /api/query` - Primary AI query endpoint (streaming responses)
- `GET /api/health` - Health check
- `GET /api/status` - System status with rate limits

### Admin Endpoints (Session Protected)
- `POST /api/admin/auth/login` - Admin login
- `GET /api/admin/stats/overview` - Dashboard statistics
- `GET /admin/api/settings/*` - Settings management (API keys, followup questions, etc.)
- `POST /admin/api/refresh` - Trigger knowledge base refresh
- `GET /api/query-logs` - Query logs with analytics

## Architecture

### Unified Smart Retriever (Zero Configuration!)
- ✅ **Auto-discovery**: Finds all content from directories automatically
- ✅ **Smart routing**: Understands query intent and routes appropriately
- ✅ **Content detection**: Categorizes technical, experience, creative content
- ✅ **No configuration**: Just drop files in directories

### Project Structure
```
backend/core/       # Core business logic (key files)
├── unified_retriever.py       # Main smart retriever system
├── smart_query_handler.py     # Query processing and routing
├── config.py                  # Centralized configuration
├── admin_auth.py              # Admin authentication
├── settings_manager.py        # Settings management
└── query_logger.py            # SQLite-based query logging

backend/knowledge/  # Auto-indexed content
├── *.md, *.pdf, *.json       # Any content automatically indexed!

admin/frontend/     # Vue.js admin dashboard
├── src/components/
├── src/views/
└── src/stores/     # Pinia state management
```

## Core Files
- `backend/core/unified_retriever.py` - **Main**: Smart auto-discovery system
- `backend/core/config.py` - **Primary**: Centralized configuration
- `backend/core/admin_auth.py` - Admin authentication
- `backend/core/settings_manager.py` - Settings management
- `pyproject.toml` - Python project configuration
- `Makefile` - Development commands

## Development Guidelines

### Adding Content (Zero Configuration!)
1. **Text content**: Drop files in `backend/knowledge/` or `public/`
   - Supports: `.md`, `.pdf`, `.json`, `.txt`, `.html`, `.docx`
   - Automatically indexed and searchable!

2. **Illustrations**: Add to `backend/knowledge/illustrations.json`
3. **Restart backend**: Content auto-discovered on startup

### Code Standards
1. **Format with Black** before committing (line length: 120)
2. **Sort imports with isort** (profile: black)
3. **Use type hints** for functions
4. **Follow existing patterns** in codebase

## Environment Variables
- `ANTHROPIC_API_KEY` - Required for Claude API access
- `GOOGLE_API_KEY` - Required for Gemini API access
- `FORCE_REBUILD_DATA=true` - Force rebuild vector indices

## Admin Dashboard

### Access
- Frontend: `npm run admin:frontend` (Vue.js + Vuetify)
- Backend: Integrated with main app at http://localhost:8000
- Authentication: Session-based with secure cookies

### Icon Usage (Vuetify + MDI)
**Always use icon aliases with `$` prefix to avoid console errors:**
```vue
<v-icon>$dashboard</v-icon>  <!-- ✅ CORRECT -->
<v-icon>mdi-home</v-icon>    <!-- ❌ WRONG - causes SVG errors -->
```

## Database Architecture
- **Query logs**: `/backend/logs/rag_monitoring.db` (SQLite)
- **Admin data**: `/backend/logs/admin_monitoring.db` (SQLite)
- **Sessions**: `/backend/logs/auth_sessions.db` (SQLite)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickberens360) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
