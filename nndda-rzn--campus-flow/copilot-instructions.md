## campus-flow

> Full-stack academic service platform: Go microservices + Next.js 16 frontend + PostgreSQL + RabbitMQ + gRPC + Protocol Buffers, organized as a Go workspace monorepo.

# CampusFlow Agent Instructions

Full-stack academic service platform: Go microservices + Next.js 16 frontend + PostgreSQL + RabbitMQ + gRPC + Protocol Buffers, organized as a Go workspace monorepo.

## Repository Map

- `apps/services/{auth,academic,file,notification,reporting}-service/` and `api-gateway/` — six Go modules wired by `go.work`.
- `apps/web/` — Next.js 16 + React 19 + Tailwind v4 frontend (single Node package).
- `proto/` — `.proto` sources. `proto/gen/` is its own Go module that the workspace `use`s.
- `db/<service>/migrations/` — goose SQL files with `-- +goose Up` / `-- +goose Down` markers, numbered, **per service database** (`auth_db`, `academic_db`, `file_db`, `notification_db`, `reporting_db`).
- `infra/docker/` — multi-stage Dockerfiles. `infra/compose/docker-compose.full.yml` brings up the entire stack. `docker-compose.yml` at root brings up Postgres + RabbitMQ only.
- `scripts/` — orchestrators (`gen-proto.ps1`, `migrate-all.{ps1,sh}`, `run-all.{ps1,sh}`).
- `.github/workflows/ci.yml` — single CI workflow with parallel `backend` + `web` jobs.

## Token & Context Policy

This repo is large; do not bulk-load it.

- Read only files relevant to the current task. Do not pre-emptively scan all services or all migrations.
- Skip generated files unless explicitly asked: `proto/gen/**/*.pb.go`, `proto/gen/**/*_grpc.pb.go`, `apps/web/.next/`, `apps/web/node_modules/`, `.opencode/node_modules/`, `storage/uploads/`.
- Before reading >5 files, state which files and why. Summarize before large or cross-service edits.

## Critical Build Order

Generated protobuf bindings are **gitignored** (`.gitignore`: `*.pb.go`, `*_grpc.pb.go`). They must exist before any Go build, including CI.

1. `.\scripts\gen-proto.ps1` (Windows-only; on Linux/CI replicate the protoc invocation it contains — see `.github/workflows/ci.yml` "Generate proto bindings").
2. `go build ./apps/services/<svc>/...` (per service; `go build ./...` from root **does not work** because of `go.work`).
3. Tests live only in `academic-service/internal/{service,repository}/` + `cmd/server/` and `file-service/internal/service/`. Other services have no `_test.go` files.

If a `.proto` is edited, regen with `gen-proto.ps1` and rebuild every service that imports the affected package.

## Frontend Quirks (Next.js 16)

- `apps/web/AGENTS.md` warns this Next.js version has breaking changes from training data. Consult `apps/web/node_modules/next/dist/docs/` before writing routing, server-action, or `useSearchParams` code.
- Any client component calling `useSearchParams()` must be wrapped in `<Suspense>` or production build (`npm run build`) fails on static export. Pattern used: `<ProtectedPage>...<Suspense fallback={null}><PageContent /></Suspense>...`.
- `npm run lint` already shows pre-existing errors in some pages (`set-state-in-effect` in `file-section.tsx`, `use-pagination.ts`, supervisor admin/head pages). Don't treat them as regressions; only flag new errors.
- `apps/web/.env.local` needs `NEXT_PUBLIC_API_BASE_URL=http://localhost:8080` for the API client to reach the gateway.

## Service Architecture Conventions

- **Layout per service**: `cmd/server/main.go` -> `internal/{config,handler,service,repository,model,messaging,worker}/`. Don't cross layers.
- **Health probes**: every Go service exposes `/healthz` (liveness) + `/readyz` (db ping + drain flag) on a dedicated port (`auth=50061`, `academic=50062`, `file=50063`, `notification=50064`, `reporting=50065`). gRPC ports stay clean for traffic.
- **Graceful shutdown**: every `main.go` uses `signal.NotifyContext(SIGINT, SIGTERM)` and `grpc.GracefulStop` with 15s drain. New services must follow this pattern.
- **Outbox pattern**: `auth_db`, `academic_db`, `file_db` each have their own `outbox_events` table + worker that ticks every 3s. Writes that produce events must INSERT into outbox in the **same transaction** as the state change. Consumers (notification, reporting) use an `inbox_events` table for SHA-based dedupe via `delivery.MessageId`.
- **Inter-service calls**: only via gRPC through generated proto clients. Never query another service's database directly.
- **Multi-tenancy (FR-277)**: non-SUPER_ADMIN reads must filter by `user_department_scopes`. SUPER_ADMIN bypasses scope.
- **Academic year (FR-278)**: only one `academic_years` row is `is_active = TRUE` (partial unique index `one_active_academic_year`). New requests auto-stamp `academic_year_id` from the active row inside the same transaction.

## Local Run / Migrate (Windows-first)

```powershell
docker compose up -d                    # postgres + rabbitmq
.\scripts\migrate-all.ps1                # apply all migrations across 5 DBs
.\scripts\run-all.ps1                    # spawns each service in its own pwsh window
```

Linux/macOS uses the matching `*.sh` scripts. There is **no `gen-proto.sh`** — replicate the protoc command from CI when needed.

`migrate-all.ps1` honors `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`, `DB_SSLMODE` env overrides. `run-all.ps1` spawns child windows; use `stop-all.ps1` (port-based kill) to cleanly tear down.

## Environment Variables That Matter

| Var | Default | Service |
| --- | --- | --- |
| `DATABASE_URL` | `postgres://campusflow:campusflow_password@127.0.0.1:5432/<db>?sslmode=disable` | all backend services |
| `RABBITMQ_URL` | `amqp://campusflow:campusflow_password@127.0.0.1:5672/` | auth, academic, file, notification, reporting |
| `JWT_SECRET` | `campusflow_dev_secret_change_me` | auth-service |
| `AUTH_SERVICE_ADDR` | `127.0.0.1:50051` | notification-service (Kaprodi broadcast lookup) |
| `MAX_FILE_SIZE_BYTES` | `10485760` | file-service |
| `ALLOWED_MIME_TYPES` | comma list, see `file-service/internal/config` | file-service |
| `SMTP_HOST` | empty -> stub mailer (logs to stdout) | auth-service forgot-password |

When `SMTP_HOST` is unset, `auth-service` uses `stubSender` and prints reset emails to stdout. To test forgot-password end-to-end without SMTP, read the worker logs for the link.

## Conventions

- **Commit style**: Conventional Commits (`feat(scope): ...`, `fix(web): ...`, `chore: ...`). Match the existing log; keep messages multi-line and explanatory.
- **Workspace boundary**: don't introduce a shared internal package between services (`internal/` is per-module). Helpers like `mail.Sender` or `worker.StartOutboxPublisher` are duplicated intentionally.
- **State machine**: academic + supervisor request status transitions live in `academic-service/internal/service/transitions.go` as a single source of truth. `CanTransition(table, from, to)` and `AllowedFrom(table, to)` are used everywhere; do not inline ad-hoc transition checks.
- **Don't commit empty AGENTS.md** at repo root by accident — there has been one regression of this. Tooling sometimes regenerates the file; only commit it when the content is meaningful.

## Pre-existing Annoyances (Don't "Fix" Without Asking)

- LF/CRLF warnings on every commit on Windows. Repo has no `.gitattributes` enforcing one or the other.
- Some frontend lint errors (`react-hooks/set-state-in-effect`) exist on legacy pages and ride along with each push. They are tracked but intentionally untouched to keep diffs focused.
- `proto/gen/go.mod` is its own Go module included via `go.work use`. Don't merge it back.

## When in Doubt

- Trust executable sources (`go.work`, `package.json`, `ci.yml`, `Dockerfile`s, migration files) over prose in `README.md` or `docs/`.
- For Next.js 16 specifics, read `apps/web/node_modules/next/dist/docs/` before guessing.
- For request workflow rules (verify, revise, reject, complete, assign), inspect `transitions.go` first.

---
> Source: [nndda-rzn/campus-flow](https://github.com/nndda-rzn/campus-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
