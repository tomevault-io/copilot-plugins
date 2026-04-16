## stratalog

> StrataLog is a game logging service with developer tools (browser, playground, documentation, stats). It provides a production API for game log submission and a developer console for viewing, testing, and managing logs.

# CLAUDE.md - StrataLog Project Instructions

## Overview

StrataLog is a game logging service with developer tools (browser, playground, documentation, stats). It provides a production API for game log submission and a developer console for viewing, testing, and managing logs.

## Key Patterns

### Feature-Based Architecture

Each feature is in `internal/app/features/<feature>/` with:
- Handler struct with dependencies
- Routes function returning http.Handler
- View models (VMs) for templates

### Store Pattern

MongoDB stores in `internal/app/store/<entity>/` with:
- `New(db *mongo.Database) *Store`
- CRUD methods with context.Context first parameter
- Input/Update structs for complex operations

### View Data

All templates receive a BaseVM with common fields:
- SiteName, Title, BackURL, CurrentPath
- IsLoggedIn, Role, UserName
- LogoURL, FooterHTML

Use `viewdata.New(r)` for minimal init or `viewdata.NewBaseVM(r, db, title, backURL)` for full settings.

### Auth

- `auth.CurrentUser(r)` returns `(*SessionUser, bool)`
- `sessionMgr.RequireAuth` middleware for authenticated routes
- `sessionMgr.RequireRole("admin")` for role-based access
- API endpoints use Bearer token authentication

## Key Features

### Log API
- POST `/api/log/submit`: Submit single or batch log entries (Bearer token auth)
- GET `/api/log/list?game=<name>`: Query logs with filters (Bearer token auth)
- Legacy: POST/GET `/logs` (backward compatible)

### Public Endpoints
- GET `/logs/view?game=<name>`: HTML view of logs (no auth)
- GET `/logs/download?game=<name>`: JSON download (no auth)

### Log Browser (`/console/api/logs`)
- View, search, filter logs by game/player/event type
- Delete operations (admin only)

### Playground & Documentation
- Interactive API testing
- API reference documentation

## Configuration

Environment variables use `STRATALOG_` prefix. See `bootstrap/config.go` and `bootstrap/appconfig.go`.

Key config:
- `STRATALOG_API_KEY`: Bearer token for API authentication
- `STRATALOG_MAX_BATCH_SIZE`: Max entries in batch submission (default: 100)
- `STRATALOG_MAX_BODY_SIZE`: Max request body size (default: 1MB)

## Common Commands

```bash
make build    # Build
make run      # Build and run
make test     # Run tests
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dalemusser) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
