## arendi

> - `backend/` contains the Go API server, WebSocket hub, migrations, and Swagger docs.

# Repository Guidelines

## Project Structure & Module Organization

- `backend/` contains the Go API server, WebSocket hub, migrations, and Swagger docs.
- `frontend/` contains the React + TypeScript web app (Vite).
- `docs/` holds planning and architecture notes.
- `mobile/` is referenced in docs but not present yet; iOS work is planned for later phases.

## Build, Test, and Development Commands

Backend (run from `backend/`):
- `make run` to start the API server.
- `make test` or `go test ./...` to run all backend tests.
- `make test-cover` to generate `coverage.html`.
- `make swagger` to regenerate OpenAPI docs in `backend/docs/`.
- `make migrate-up` / `make migrate-down` for database migrations (needs `DATABASE_URL`).

Frontend (run from `frontend/`):
- `npm run dev` for local development at `http://localhost:5173`.
- `npm run build` to build production assets in `frontend/dist/`.
- `npm run lint` and `npm run format` for code quality and formatting.
- `npm run type-check` for TypeScript checks.

## Coding Style & Naming Conventions

- Go: standard `gofmt` formatting, `golangci-lint` if available.
- TypeScript: ESLint + Prettier, strict TS config.
- Branch names: `feature/`, `bugfix/`, or `hotfix/` prefixes.
- Commit messages: clear, descriptive, imperative mood (e.g., "Add user auth endpoints").

## Testing Guidelines

- Backend: Go `testing` package; unit and integration tests required for API endpoints.
- Frontend: tests planned (Vitest + React Testing Library per project guidelines).
- Coverage target: aim for 80%+ when tests exist; update as the test suite grows.

## Commit & Pull Request Guidelines

- Link PRs to GitHub issues; describe scope and testing performed.
- Include screenshots or GIFs for UI changes.
- PRs require review and passing tests before merge.

## GitHub Interaction Preferences

- When creating issues, always add them to Project 3: `https://github.com/users/matdemers1/projects/3`.
- Default new issue metadata (unless specified otherwise): milestone `MVP - Phase 1`, labels `backend`, `enhancement`, `priority:medium`, `MVP`.
- Always update Swagger (`make swagger` in `backend/`) after backend changes.
- Commit message format: multi-section, bullet-heavy template like the provided example; omit any `Co-Authored-By` footer.
- Workflow for issues: set to In Progress → implement changes → run lint → fix lint issues → run tests → fix test failures → provide testing steps → wait for user verification → then commit/push and close issue.
- After committing, push to `origin main` and close the related issue with a brief summary comment.

## Security & Configuration Tips

- Use `.env` files in `backend/` and `frontend/` for local configuration.
- Never commit secrets; keep JWT secrets and database credentials local.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matdemers1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/matdemers1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
