## bandeira

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bandeira** is a self-hosted, open-source feature flag service built with Go. Ships as a single binary + SQLite. Forked from Pagode (Go/Echo/Ent/InertiaJS/React starter kit).

## Key Technologies

- **Backend**: Go 1.26, Echo v4 web framework, Ent ORM, SQLite
- **Frontend**: React 19, TypeScript, InertiaJS, Tailwind CSS v4, shadcn/ui
- **Build Tools**: Vite (frontend), Air (Go hot reload)

## Common Development Commands

- `make run` - Start the application (default: http://localhost:8080)
- `make watch` - Start with hot reload (requires `make air-install` first)
- `make test` - Run all tests
- `go build -o /dev/null ./cmd/web` - Test compilation without generating binaries
- `make ent-gen` - Generate Ent ORM code (run after schema changes)
- `make ent-new name=Entity` - Create new Ent entity

## Architecture Overview

### Service Container Pattern
All application dependencies are managed through a service container (`pkg/services/container.go`):
- Database client, Cache, Configuration, Inertia
- Use `services.NewContainer()` to initialize, `container.Shutdown()` to cleanup
- Available in handler `Init()` methods for dependency injection

### Directory Structure
- `cmd/web/` - Application entry point
- `pkg/handlers/` - HTTP request handlers with route registration
- `pkg/middleware/` - Custom middleware
- `pkg/services/` - Service container and business logic
- `ent/schema/` - Ent ORM entity definitions
- `resources/js/Pages/` - InertiaJS React page components
- `resources/js/components/` - Reusable React components
- `config/` - Configuration management (YAML + env overrides)

### Frontend Architecture
- **InertiaJS** bridges Go controllers with React components
- Server-side routing with client-side interactivity
- Pages in `resources/js/Pages/`, components in `resources/js/components/`

## Development Patterns

### Adding New Routes
1. Create handler in `pkg/handlers/`
2. Embed required services in handler struct
3. Use `Register(new(YourHandler))` in `init()`
4. Implement `Init(c *services.Container)` for dependency injection
5. Add routes in `Routes(g *echo.Group)` method with named routes

### Testing
- Use `config.SwitchEnvironment(config.EnvTest)` before creating container
- **IMPORTANT**: Always run `make test` after making changes

## Configuration

Configuration uses Viper with `config/config.yaml` as base. Environment variables override config using `BANDEIRA_` prefix (e.g., `BANDEIRA_HTTP_PORT` overrides `http.port`, `BANDEIRA_AUTH_ADMINEMAIL` overrides `auth.adminEmail`).

## Authentication & RBAC

- Email/password login with bcrypt hashing
- Three roles: `admin` (full access), `editor` (manage flags/projects, no user mgmt), `viewer` (read-only)
- First admin user seeded from `BANDEIRA_AUTH_ADMINEMAIL` + `BANDEIRA_AUTH_ADMINPASSWORD` env vars
- Session stores user ID via `pkg/session/auth.go`
- `RequireRole(orm, roles...)` middleware enforces role-based access on routes
- `InertiaProps` middleware loads real user from DB and shares `role` in frontend props

## File Generation

When modifying Ent schemas:
1. Edit schema files in `ent/schema/`
2. Run `make ent-gen` to regenerate all Ent code

## Important Notes

- CSRF protection enabled by default for non-GET requests
- Static files served from `static/` directory at `/files/` URL prefix
- Cache is in-memory (otter library)

---
> Source: [felipekafuri/bandeira](https://github.com/felipekafuri/bandeira) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
