## finance-tracker

> Guidance for Claude Code working in this repository.

# CLAUDE.md

Guidance for Claude Code working in this repository.

## Repo Structure

Monorepo: Go API in `/backend`, React app in `/frontend`.

## Commands

```sh
# Infrastructure
docker compose up -d db          # Start PostgreSQL
make up                          # Full stack (docker compose up -d --build)
make down                        # docker compose down
# Note: migrations run automatically on backend startup (cmd/api/main.go).

# Backend
make dev-backend                 # Run API locally
make sqlc                        # Regenerate sqlc code after editing queries/*.sql
cd backend && go test ./...      # Run all tests
cd backend && go test ./internal/service/ -run TestFunctionName  # Single test

# Production
make prod-build                  # Build prod images
make prod-push                   # Push backend & frontend to registry
make prod-deploy                 # Pull & up on VPS

# Frontend
make dev-frontend                # Vite dev server (proxies /api to :8080)
cd frontend && npm run build     # Type-check + production build
cd frontend && npm run lint      # ESLint
cd frontend && npm run check:types  # TypeScript check only
cd frontend && npm test          # Run all tests (Vitest)
```

## Environment Variables

Required: `DATABASE_URL` (postgres connection string), `JWT_SECRET` (HMAC key, 32+ chars), `INVITE_CODES` (comma-separated list of valid registration invite codes). Optional: `PORT` (default 8080), `EXCHANGE_RATE_SYNC_MODE` (`"endpoint"` default or `"background"`), `EXCHANGE_RATE_SYNC_TOKEN` (static token for sync endpoint), `COOKIE_SECURE` (default `true`; set to `false` for local HTTP dev so the refresh cookie is sent over http://), `BASE_PATH` (URL prefix the app is served under, default `/`; e.g. `/finance/` when hosted behind a reverse proxy — docker-compose feeds this to the frontend build as `VITE_BASE_PATH` and to the backend runtime as `BASE_PATH`, and the refresh cookie's Path is derived from it). Production: `IMAGE_REGISTRY` (container registry prefix, e.g. `ghcr.io/username`).

## Specifications

Frontend-specific guidance in `frontend/CLAUDE.md`.
Backend-specific guidance in `backend/CLAUDE.md`.

## Code Quality

- When fixing linter or type errors, prefer proper configuration changes (e.g., ESLint rules, tsconfig settings) over inline disables or quick-fix suppressions. Always ask before adding eslint-disable comments.

## Conventions

- When changes touch both backend and frontend, always run both `cd backend && go test ./...` and `cd frontend && npm test` before reporting completion.
- When changing backend error codes, update all of: handler response, `frontend/src/lib/error-mapping.ts`, `frontend/src/lib/__tests__/error-mapping.test.ts`, `backend/docs/API.md` error table, and `backend/docs/ARCHITECTURE.md` error table.
- API changes must be reflected in [backend/docs/API.md](backend/docs/API.md)
- Backend architecture changes must be reflected in [backend/docs/ARCHITECTURE.md](backend/docs/ARCHITECTURE.md)
- Backend feature changes must be reflected in [backend/README.md](backend/README.md)
- When adding new environment variables, update all of: the `Environment Variables` section in this file, `README.md`, `.env.example`, and `docker-compose.yml`.
- When the user asks for a code review or simplification suggestions, present findings first and wait for user approval before implementing changes. Do not continue fixing iteratively without checking in.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/berrunder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
