## 360-photo-editor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**One-liner:** Google Photos-style AI-powered gallery and photo editor with Flutter frontend and FastAPI backend.

This is a 360 Photo Editor application consisting of:

1. **Flutter Frontend** (`ai_image_editor/`) - Cross-platform mobile app
2. **Python Backend** (`ai_image_editor_backend/`) - FastAPI-based API server

## Quick Context

| Component | Technology | Key Pattern |
|-----------|------------|-------------|
| Frontend | Flutter 3.x + Dart 3 | Clean Architecture + GetX |
| Backend | Python FastAPI + async SQLAlchemy 2.0 | Modular routers |
| Auth | Supabase JWT | Middleware validation |
| AI | Google Gemini API | Image generation + NL parsing |
| Storage | Supabase Storage | Authenticated downloads |

## Common Commands

### Flutter Frontend

```bash
cd ai_image_editor

# Run the app (requires dart-define for API URL)
flutter run --dart-define=AI_IMAGE_EDITOR_API_BASE_URL=http://10.0.2.2:8000 --dart-define=SUPABASE_ACCESS_TOKEN=<token>

# Run tests
flutter test

# Run a specific test file
flutter test test/presentation/controllers/editor_controller_test.dart

# Run a single test by name
flutter test --name "test name"

# Analyze code
flutter analyze

# Build for production
flutter build apk
flutter build ios
```

### Python Backend

```bash
cd ai_image_editor_backend

# Install dependencies
pip install -e .

# Run the server (requires .env file)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Run with specific environment
APP_ENV=development uvicorn app.main:app --reload

# Health check
curl http://localhost:8000/health
```

## Architecture

### Flutter Frontend (Clean Architecture)

The app follows Clean Architecture with three layers:

```
lib/
├── core/           # Shared infrastructure
│   ├── di/         # Dependency injection (GetX)
│   ├── network/    # Dio HTTP client
│   ├── storage/    # Isar + SharedPreferences
│   ├── routes/     # GetX routing
│   └── auth/       # JWT token management
├── data/           # Data layer
│   ├── datasources/# Remote and local data sources
│   ├── models/     # DTOs / data models
│   └── repositories/ # Repository implementations
├── domain/         # Domain layer
│   ├── entities/   # Business entities (ai_jobs, edit_ops, filters, templates)
│   ├── repositories/ # Repository interfaces
│   └── usecases/   # Use cases
└── presentation/   # Presentation layer
    ├── controllers/ # GetX controllers
    ├── pages/       # Screens
    ├── widgets/     # Reusable widgets
    └── bindings/    # GetX dependency bindings
```

**Key Patterns:**
- State management: GetX (reactive programming with `Rx<T>` variables)
- Navigation: GetX routing with `AppPages` and `AppRoutes`
- Dependency injection: Manual registration in `DependencyInjection.init()`
- HTTP client: Dio with interceptors for logging (debug mode only)
- Compile-time config: `--dart-define` for API URLs and auth tokens

### Python Backend (FastAPI)

```
app/
├── main.py         # FastAPI app entry point
├── config.py       # Pydantic settings from .env
├── auth/           # JWT verification middleware (Supabase)
├── db/             # SQLAlchemy 2.0 models and session
├── nl/             # Natural language parsing (Gemini integration)
├── ai_jobs/        # AI image generation jobs
├── templates/      # Template presets
├── usage/          # Rate limiting and quota management
└── storage/        # File storage operations
```

**Key Patterns:**
- Database: SQLAlchemy 2.0 with asyncpg
- Auth: Supabase JWT verification middleware
- AI: Google Gemini API for NL parsing and image generation
- Config: Pydantic Settings with `.env` file
- Rate limiting: Daily/weekly quotas stored in PostgreSQL with row locking

## Critical File Locations

### Frontend

| Purpose | Path |
|---------|------|
| Main entry | `lib/main.dart` |
| Routes | `lib/core/routes/app_pages.dart` |
| DI setup | `lib/core/di/dependency_injection.dart` |
| Gallery screen (home) | `lib/presentation/pages/gallery/gallery_screen.dart` |
| Image viewer | `lib/presentation/pages/gallery/image_viewer_screen.dart` |
| Editor screen | `lib/presentation/pages/editor/editor_screen.dart` |
| Create studio | `lib/presentation/pages/create/create_screen.dart` |
| Profile screen | `lib/presentation/pages/profile/profile_screen.dart` |
| Gallery controller | `lib/presentation/controllers/gallery_controller.dart` |
| Edit operations | `lib/domain/entities/edit_ops/edit_op.dart` |
| AI jobs | `lib/domain/entities/ai_jobs/ai_job.dart` |
| API client | `lib/data/datasources/remote/remote_datasource.dart` |

### Backend

| Purpose | Path |
|---------|------|
| Entry point | `app/main.py` |
| Config | `app/config.py` |
| AI jobs router | `app/ai_jobs/router.py` |
| AI jobs service | `app/ai_jobs/service.py` |
| NL router | `app/nl/router.py` |
| Gemini client | `app/nl/gemini_client.py` |
| Auth middleware | `app/auth/middleware.py` |
| Usage service | `app/usage/service.py` |

## Configuration

### Frontend Compile-Time Variables

- `AI_IMAGE_EDITOR_API_BASE_URL` - Backend API base URL
- `SUPABASE_ACCESS_TOKEN` - User JWT access token

### Backend Environment Variables

Required secrets (in `.env`):
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY`
- `GEMINI_API_KEY`

App settings (in `.env`):
- `DATABASE_URL` - PostgreSQL connection string
- `NL_PARSE_DAILY_LIMIT_FREE` - Default: 10
- `AI_DAILY_CREDITS_FREE` - Default: 5
- `TEMPLATES_WEEKLY_LIMIT_FREE` - Default: 3
- `AI_JOBS_OUTPUT_BUCKET` - Default: edited-images

## Key Entities

### AI Job Types

| Type | Credits | Description |
|------|---------|-------------|
| `bgRemove` | 1 | Remove image background |
| `upscale2x` | 2 | 2x image upscaling |
| `autoEnhance` | 1 | Auto color correction |
| `objectErase` | 2 | Remove unwanted objects |
| `aiRemaster` | 3 | Restore old/damaged photos |

### Edit Operation Types

`crop`, `rotate`, `flip`, `resize`, `adjust`, `filter`, `textOverlay`

### Filter Presets

`clean`, `vivid`, `warm_glow`, `cool_mist`, `cinematic`, `urban_matte`, `noir`, `emerald`, `golden_hour`, `faded_film`, `peach_pop`, `night_drive`

### Usage Tracking

- Daily counters for NL parsing and AI jobs
- Weekly counters for template usage
- Per-user tracking with Supabase user IDs

## Free vs Premium Limits

| Feature | Free | Premium |
|---------|------|---------|
| Natural Language edits/day | 10 | Unlimited |
| AI job credits/day | 5 | 40 |
| Templates/week | 3 | Unlimited |
| **Price** | Free | ₹99/month or ₹499/year |

## Common Tasks

### Adding a New Edit Operation

1. Add to `EditOpType` enum in `lib/domain/entities/edit_ops/edit_op.dart`
2. Add validation in backend `app/nl/edit_ops_validator.py`
3. Add UI panel in `lib/presentation/widgets/`
4. Wire up in `EditorController`

### Adding a New AI Job Type

1. Add to `AIJobType` enum in `lib/domain/entities/ai_jobs/ai_job.dart`
2. Add credit cost in backend `app/ai_jobs/constants.py`
3. Add processing logic in `app/ai_jobs/service.py`
4. Add UI button in `lib/presentation/widgets/ai_enhance_panel.dart`
5. Wire up in `EditorController.runAiJob()`

### Adding a New API Endpoint

1. Add route in appropriate `router.py`
2. Define schemas in `schemas.py`
3. Add service logic in `service.py`
4. Add datasource method in frontend `lib/data/datasources/remote/remote_datasource.dart`
5. Add use case in `lib/domain/usecases/`

## Implementation Notes

- **Navigation**: Bottom navigation bar with Photos (Gallery), Create, Profile. Gallery is the home screen.
- **State Management**: Uses GetX reactive pattern. Controllers extend `GetxController`, use `Rx<T>` for reactive vars.
- **Undo/Redo**: `EditOpsStack` class manages operation history with max 10 steps.
- **Image Preview**: Uses ColorFiltered widgets with color matrices for adjustments.
- **AI Enhancement Flow**: AI analyzes image → suggests improvements → user selects → generates new stacked image.
- **AI Job Polling**: Frontend polls every 900ms for up to 21.6 seconds.
- **Rate Limiting**: Backend uses database row locking (`with_for_update()`) for credit consumption.
- **NL Parsing**: Gemini 1.5 Flash parses text to JSON edit operations with confidence score.
- **Auth**: JWT from Supabase, middleware validates on every request except `/health`.
- **Permissions**: iOS 14+ Limited Photos Access is fully supported.

## When Editing

- Prefer using existing use cases and repositories
- Follow the Clean Architecture pattern (don't bypass layers)
- Use existing widgets and patterns from the codebase
- Maintain the dark theme aesthetic (Color 0xFF0A0C11)
- Reuse existing enums and constants rather than creating new ones

## Documentation

Full documentation available in `/docs` folder:
- `docs/architecture/` - System design
- `docs/features/` - Feature details
- `docs/flows/` - User flows
- `docs/api/` - API reference
- `docs/development/` - Setup guides

---
> Source: [360ghar/360-photo-editor](https://github.com/360ghar/360-photo-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
