## stratasave

> Stratasave is a Go web application starter/template for building monitor-able services. It provides authentication, user management, settings, and dynamic pages out of the box.

# CLAUDE.md - Stratasave Project Instructions

## Overview

Stratasave is a Go web application starter/template for building monitor-able services. It provides authentication, user management, settings, and dynamic pages out of the box.

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

## Adding Features

1. Create `internal/app/features/<name>/<name>.go`
2. Define Handler, NewHandler, Routes
3. Mount in `bootstrap/routes.go`
4. Create templates in `resources/templates/<name>/`

## Configuration

Environment variables use `STRATASAVE_` prefix. See `bootstrap/config.go` and `bootstrap/appconfig.go`.

## Common Commands

```bash
make build    # Build
make run      # Build and run
make test     # Run tests
```

---
> Source: [dalemusser/stratasave](https://github.com/dalemusser/stratasave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
