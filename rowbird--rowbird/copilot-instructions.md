## rowbird

> Guidance for AI coding agents (and humans) working on this repository. Read

# AGENTS.md: Rowbird

Guidance for AI coding agents (and humans) working on this repository. Read
[CONTRIBUTING.md](CONTRIBUTING.md) too.

Rowbird is a self-hosted, open source service that runs SQL queries on a schedule and delivers the
results, formatted as CSV, Excel, PDF, JSON or inline tables, to email, Telegram, Slack, Discord,
webhooks, Uptime Kuma, S3 and more. Conditions turn reports into data alerts.

Tagline: **"Your SQL results, delivered."** · Website: https://rowbird.dev · Docs: https://docs.rowbird.dev

## Writing style

- **Everything in the repository is in English**: code, identifiers, comments, commit messages,
  docs, ADRs, OpenAPI descriptions, log messages and error codes. The only exception is the content
  of the `pt-BR` locale files.
- **Write in natural, clear prose**, like a senior engineer explaining something to a colleague.
- **No emojis**, anywhere (code, comments, commits, docs).
- **No em dashes or en dashes** (the long dash characters). Use commas, colons, parentheses or
  separate sentences instead. Regular hyphens in compound words and code are fine.
- Avoid filler, hype and excessive bold. Be direct and specific.

---

## Sources of truth (read before working)

| What | Where |
|---|---|
| Product & behavior spec | `docs/spec/*.md` (one file per topic) |
| Architecture decisions | `docs/adr/*.md` (do NOT silently contradict an accepted ADR) |
| Roadmap | `docs/plan/ROADMAP.md` (what ships in v1 and what comes next) |
| HTTP API contract | `api/openapi.yaml` (spec-first, everything else is generated from it) |

If the spec is ambiguous, prefer the simplest behavior consistent with it and **write down the
decision** (update the spec, or add an ADR if it is architectural). If the spec seems wrong, stop and
ask instead of improvising.

## Stack

- **Backend:** Go (latest stable), single binary. Router `chi`, logging `log/slog`, OpenAPI server
  stubs via `oapi-codegen` (strict server).
- **Internal store:** SQLite by default (`modernc.org/sqlite`, pure Go, no CGO), PostgreSQL optional
  (`pgx`). Migrations per dialect. Every table carries `workspace_id`.
- **Frontend:** Vue 3 + TypeScript + Vite, `<script setup lang="ts">`, Pinia, Vue Router, vue-i18n,
  Tailwind + shadcn-vue, CodeMirror 6 for SQL. API client generated with `openapi-typescript` +
  `openapi-fetch`. Built assets are embedded in the Go binary via `embed`.
- **Key libs:** excelize (XLSX), maroto v2 (PDF), robfig/cron v3 (cron parsing only), minio-go (S3),
  x/crypto (argon2id, ssh), pquerna/otp (TOTP), go-webauthn (passkeys), coreos/go-oidc,
  prometheus/client_golang, a Mustache implementation for message templates.
- **Tests:** Go `testing` + testcontainers-go; Vitest; Playwright. **Docs:** VitePress. **Release:** GoReleaser.

## Repository layout

```
cmd/rowbird/            main.go: CLI entrypoint (serve, migrate, backup, apply, ...)
api/openapi.yaml        API contract (source of truth)
internal/
  api/                  HTTP handlers implementing generated interfaces, middleware
  app/                  wiring / dependency injection
  auth/                 sessions, passwords, TOTP, passkeys, OIDC, API keys
  store/                repositories + migrations (sqlite/, postgres/); workspace scoping lives here
  scheduler/            due-report polling, claiming, next_run_at
  runner/               worker pool, run lifecycle, retries
  plugin/               registry + shared contracts (Connector, Formatter, Condition, Destination, ...)
  connector/<driver>/   postgres, mysql, mssql, sqlite (+ ssh tunnel helper)
  format/<name>/        csv, xlsx, json, pdf, inline (html, markdown, text)
  condition/            built-in conditions
  destination/<name>/   email, telegram, slack, discord, webhook, uptimekuma, s3
  storage/<name>/       artifact storage: local, s3
  ai/<provider>/        openai, anthropic, gemini, ollama, openai-compatible
  notify/               in-app notifications, system alerts, grouping, heartbeat
  gitops/               YAML config-as-code: export, import, apply, diff
  crypto/               AES-256-GCM secret encryption, key rotation
  params/               query parameter parsing + binding
  i18n/                 server-side message catalogs
  metrics/  health/  security/
ee/                     commercial-licensed code (empty in v1, reserved)
web/                    Vue app
docs/                   spec, adr, plan (roadmap), site (VitePress, docs.rowbird.dev), assets
deploy/                 docker-compose examples, systemd unit, k8s example manifests
testdata/               fixtures, demo seed SQL
```

## Commands (maintain these in the Makefile)

```
make dev               # backend with live reload + Vite dev server (proxy /api)
make generate          # regenerate Go server stubs + TS client from api/openapi.yaml
make test              # unit tests (Go + Vitest)
make test-integration  # testcontainers: postgres, mysql, mariadb, mssql, mailpit, versitygw (S3)
make test-e2e          # Playwright against a built binary (Docker: mailpit, versitygw)
make lint              # golangci-lint + eslint + vue-tsc
make build             # web build -> embed -> single binary in ./bin/rowbird
```

## How to work

1. Read the spec files and ADRs that cover the change before writing code.
2. Deliver vertical slices: migrations + store + service + API (openapi first) + UI + tests.
3. **API changes start in `api/openapi.yaml`**, then `make generate`. Never hand-edit generated code.
4. Small commits using **Conventional Commits** (`feat(scheduler): ...`, `fix(connector/mysql): ...`).
5. Keep docs in sync: behavior change → update `docs/spec` and `docs/site`; architectural change →
   new ADR.

## Hard rules (never break these)

- **Secrets never leave encrypted storage in plain text**: not in logs, errors, API responses
  (return `"configured": true` instead), exports, or notifications. Use the log redactor.
- **Every data access is scoped by `workspace_id` in the store layer**, never ad hoc in handlers.
- **Query parameters are always bound as driver parameters**, never string-concatenated into SQL.
- **User databases are read-only by default**: read-only transactions where supported,
  server-side statement timeouts, row limits, single statement unless the connection allows more.
- **Escape all database-originated content** in HTML (emails, PDFs) and neutralize CSV/Excel
  formula injection (`= + - @` prefixes, tab/CR).
- **AI output is never executed automatically**: it is always a proposal the user applies.
- **No telemetry.** No outbound calls except those the user configured (plus the optional,
  disableable update check).
- Plugins must declare their config schema (with secret fields marked) and pass the
  **plugin conformance test suite**.
- No user-facing string is hardcoded: backend and frontend use i18n keys, with `en` and `pt-BR`.

## Go conventions

- `context.Context` first argument everywhere I/O happens; respect cancellation.
- Wrap errors with `%w`; map domain errors to stable API error codes (`connection.auth_failed`).
- Interfaces are defined where they are consumed; plugin contracts live in `internal/plugin`.
- No global mutable state except plugin registries populated at init.
- Table-driven tests; integration tests behind the `integration` build tag.
- Times in UTC internally; convert only at the edges (report timezone, UI).
- Money/decimals are never converted to float.

## Frontend conventions

- Only the generated API client talks to the backend.
- Forms for plugins are rendered from the plugin's config schema (`GET /api/v1/plugins`).
- All strings via vue-i18n (`en`, `pt-BR`). Dates/numbers formatted with the user's locale.
- Every list has an empty state that explains the concept and offers the create action.
- Destructive actions require confirmation and show impact ("used by 3 reports").

## Definition of done (every task)

- [ ] OpenAPI updated and code regenerated (if API changed)
- [ ] Unit tests + integration tests where I/O is involved; all green in CI
- [ ] UI strings in `en` and `pt-BR`
- [ ] Permissions enforced and tested; workspace scoping respected
- [ ] No secret can appear in logs/responses (add a test when handling secrets)
- [ ] Spec/ADR/ROADMAP updated

---
> Source: [rowbird/rowbird](https://github.com/rowbird/rowbird) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
