## oauth2-impl

> Guidance for AI agents working in this repo.

# AGENTS.md

Guidance for AI agents working in this repo.

## Hard rules

- **Do NOT run `serve`, `migrate`, `migrate-force`, `make dev`, `make go`, `make tw`, or any command that starts the server or touches the database.** These are for the human to run. Build/typecheck/compile edits only.
- **Do not cross the backend/frontend boundary in a single task unless the user explicitly asks for it.** Backend is `internal/v1`, `internal/app`, `internal/entities`, etc.; frontend is `internal/web/`. If a change genuinely requires both and the user didn't say so, stop and tell the user instead of doing both yourself.
- **Never fabricate library/package APIs.** Before writing code that uses any library (Gin, GORM, HTMX, Alpine.js, dig, cobra, swaggo, etc.), look up the real API via the Context7 MCP (`resolve-library-id` → `query-docs`). Do not rely on memory for signatures, generics syntax, or HTMX attributes.
- **Retain the existing project structure and layering.** Do not introduce new top-level packages or reorganize without being asked.
- **Authenticated in-app navigation must use HTMX partial swaps.** Clicking links, tabs, pagination, filters, form actions, or other controls inside the authenticated web shell must update only the relevant content region and must not reload or replace the whole document. Preserve the browser URL/history with `hx-push-url` when the destination represents a navigable page. Full-document responses remain required for direct URL loads and browser reloads; full navigation is also allowed when protocol or browser behavior requires it, such as login/logout boundaries, OAuth redirects to an OAuth Client, file downloads, or an explicit user request.

## Domain language

`CONTEXT.md` (repo root) defines the OAuth terminology to use in code, UI copy, and commits — e.g. **Authorization Server** (this service), **OAuth Client** (external app), **Consent**, **Scope**, **Authorization Grant/Denial**. Follow its "avoid" list (e.g. don't say "permission" for a protocol Scope).

## Backend / Frontend split

- **Backend** (REST API): `main.go`, `cmd/`, `internal/app/`, `internal/configs/`, `internal/container/`, `internal/entities/`, `internal/enums/`, `internal/common/`, `internal/v1/`, `pkg/`.
- **Frontend** (server-rendered UI): `internal/web/` — see its own section below.

## File naming conventions (important)

Files are split per domain, one route per file, but multiple related structs/functions may share a file when they belong to the same domain. Match existing suffixes exactly:

| Layer          | Directory                          | Suffix           | Example                |
|----------------|------------------------------------|------------------|------------------------|
| Entity         | `internal/entities/`               | `.entity.go`     | `user.entity.go`       |
| Model (DTO)    | `internal/v1/models/`              | `.model.go`      | `user.model.go`        |
| Repository     | `internal/v1/repositories/`        | `.repo.go`       | `user.repo.go`         |
| Service        | `internal/v1/services/`            | `.service.go`    | `user.service.go`      |
| Handler        | `internal/v1/handlers/`            | `.handler.go`    | `user.handler.go`      |
| Route          | `internal/v1/routes/`              | `.route.go`      | `user.route.go`        |

- A new domain (e.g. `foo`) means: `foo.entity.go`, `foo.model.go`, `foo.repo.go`, `foo.service.go`, `foo.handler.go`, `foo.route.go`. All six are usually needed.
- A file like `user.model.go` may hold multiple related types (e.g. `CreateUserModel`, `UserPageModel`). Group by domain, not one-type-per-file.
- Do not put routes in a single `routes.go`; each resource gets its own `*.route.go` and is wired up in `routes/router.go`.

## Backend architecture

- **Stack**: Go 1.26, Gin, GORM + PostgreSQL, JWT (golang-jwt/v5) + argon2id passwords, Cobra CLI, Swagger via swaggo, n8n integration.
- **Entry point**: `main.go` → `cmd.Execute()` (Cobra). Commands: `serve`, `migrate` (`--drop` to drop first).
- **Dependency injection**: `go.uber.org/dig`. Register every new repo/service/handler in `internal/container/di.go`. Repos and services are registered as interfaces via `dig.As(new(repo.IFooRepo))`. The `App` struct in `internal/app/app.go` receives the top-level handler groups (`routes.APIHandlers`, `webroutes.WebHandlers`); when adding a new domain you must: add it to the handler group struct, register in `di.go`, and wire its routes in the relevant `router.go`.
- **Generic CRUD contract**: repos implement `common.IBaseCrudRepo[TEntity, TInput, TPage]` and services implement `common.IBaseCrudService[...]` (see `internal/common/base_crud.go`). Extend the interface for domain-specific methods (e.g. `Login`, `GetPermissions`).
- **`Read` is pagination-driven via `morkid/paginate`**: the `Read` repo/service methods take a `*gin.Context` and call `paginate.New().With(stmt).Request(c.Request)` to parse pagination query params straight from the HTTP request. Therefore, when the web UI calls a `Read` service, pagination must be carried by the **browser URL query string** (e.g. HTMX `hx-get` links with the page params in the URL) — do not construct query params programmatically inside the handler.
- **GORM generics API**: this codebase uses the Go 1.26 `gorm.G[T](db)` form (e.g. `gorm.G[entities.User](r.db).Where(...).First(c)`), passing the `context.Context` (`c` / `ctx`) as the first arg. Don't switch to the non-generic `db.First(&model)` style.
- **Response envelope**: always return JSON via `common.ResultOk[T](items, msg)` / `common.ResultErr(err, msg)` (defined in `internal/common/result.go`), not raw `gin.H`.
- **Swagger annotations**: handlers carry swaggo `// @...` comments. Swagger docs live in `docs/` (gitignored, generated). Regeneration requires `make swag` — do not run it yourself; flag it to the user.
- **Config/env**: loaded via godotenv from `.env` in `internal/configs/env.go`. Required vars are in `example.env` (PORT, DB_*, JWT_SECRET, SERVER_ENV, DEFAULT_*, N8N_BASE_URL, R2_*, SMTP_*). `.env` is gitignored.

## Frontend (`internal/web/`)

- **Stack**: HTMX + Alpine.js, server-rendered HTML via Go `html/template`. Prefer HTMX partial swaps (`hx-get`/`hx-post`/`hx-swap`, OOB swaps, `hx-target`) over full page reloads or client-side state. Verify exact HTMX/Alpine attributes via Context7 before writing them. HTMX is vendored locally at `internal/web/public/scripts/htmx.min.js` (no CDN).
- **Structure** (mirror the backend's per-domain convention):
  - `internal/web/routes/` — `<name>.route.go`, wired centrally in `routes/router.go` via the `WebHandlers` struct (`dig.In`). Web routes are registered on the root engine by `webroutes.SetupWebRoutes` in `internal/app/app.go`.
  - `internal/web/handlers/` — `<name>.handler.go` (e.g. `AuthWebHandler`, `OAuthHandler`).
  - `internal/web/views/` — `.html` templates (`layout.html` is the base layout).
  - `internal/web/public/` — static assets served at `/public`.
- **Rendering** (see `handlers/renderer.go`): `renderTemplate` parses `layout.html` + the page template; `renderFragment` renders a bare partial for HTMX swaps. Templates are parsed from disk at `internal/web/views/` with paths relative to the repo root — the server must run from the repo root.
- **Partial navigation contract**: the authenticated sidebar/layout is persistent. In-shell page links must target the right-side content boundary (currently `#app-content`) with HTMX and push the requested URL into browser history. Handlers must distinguish full document requests from HTMX fragment requests (for example via `HX-Target`) and return only the requested fragment. Nested interactions must target their narrowest stable boundary (for example, pagination swaps only its registry/table region). Never use JavaScript to synthesize pagination URLs when the service reads query parameters from `*gin.Context`; render those URLs into `hx-get`/`href` attributes on the server.
- Normal anchors remain as progressive-enhancement fallbacks, but every eligible authenticated in-app click must include the HTMX attributes needed to prevent a full page reload. When adding a new authenticated page, add or extend a regression test that verifies its navigation target, partial response path, and history behavior.
- When adding a frontend page for an existing backend resource, reuse the backend services (don't reimplement data access in the web layer).
- **Do not design generic "slop" UI.** Load the `frontend-design` skill before building or restyling any UI. Avoid the common AI-generated tells: excessive padding/margin, cards that jump/translate on hover, borders that brighten on hover, overused gradients, thick borders, and heavy/overused shadows or glow effects. Aim for deliberate, restrained styling instead.

### Tailwind CSS

- **Tailwind v4 via the standalone CLI** — there is no `package.json`/Node toolchain in this repo. Don't add one.
- Source of truth: `internal/web/public/css/input.css` (`@import "tailwindcss" source("../../")` — Tailwind scans all of `internal/web/`, so classes in `.html` views **and** Go handler strings are detected automatically).
- **Never hand-edit `internal/web/public/css/output.css`** — it is generated and committed. Any custom CSS/`@theme` tokens go in `input.css`.
- Class changes require a rebuild: the human runs `make tw` (watch mode). A one-off regeneration is `tailwindcss -i ./internal/web/public/css/input.css -o ./internal/web/public/css/output.css` (drop `--watch`). Flag to the user when template edits need a rebuild; untracked new classes won't appear in `output.css` until then.

## Verification you can do

- `go build ./...` — confirm it compiles. Run this after edits; it's the safe check.
- `go vet ./...` — static checks.
- `go test ./...` — there is currently no test suite; don't assume one exists.
- Do **not** run `make dev`, `make go`, `air`, `serve`, `migrate`, or `make tw` to verify — see Hard rules.

## Other notes

- `.env`, `docs/`, and `tmp/` are gitignored. Don't commit `.env` or regenerated swagger.
- CI (`.github/workflows/docker-publish.yml`) builds and publishes the Docker image; deploy uses `docker-compose.yml` (runs `migrate` then `serve` in containers).
- Some role/permission logic is intentionally stubbed/commented out (e.g. `AssignRoles`, `GetPermissions` in the user domain) — don't "fix" it unless asked.

---
> Source: [MarcelArt/oauth2-impl](https://github.com/MarcelArt/oauth2-impl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
