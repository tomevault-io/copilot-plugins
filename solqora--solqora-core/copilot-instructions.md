## solqora-core

> AI API gateway built with Go. Aggregates 40+ upstream AI providers behind a unified API, with user management, billing, rate limiting, and admin dashboard.


# AGENTS.md — Project Conventions for Solqora Core

## Overview

AI API gateway built with Go. Aggregates 40+ upstream AI providers behind a unified API, with user management, billing, rate limiting, and admin dashboard.

## Tech Stack

- **Backend**: Go 1.25, Gin web framework, GORM v2 ORM
- **Frontend**: React 19, TypeScript, Rsbuild, Base UI, Tailwind CSS
- **Databases**: SQLite, MySQL, PostgreSQL
- **Cache**: Redis + in-memory cache
- **Auth**: JWT, WebAuthn/Passkeys, OAuth

## Architecture

Layered: Router -> Controller -> Service -> Model

```
router/        — HTTP routing (API, relay, dashboard, web)
controller/    — Request handlers
service/       — Business logic (billing, quota, relay)
model/         — Data models and DB access (GORM)
relay/         — AI provider adapters
middleware/    — Auth, rate limiting, CORS, logging
web/           — Frontend (React 19)
```

## Billing & Solana Integration

- `service/topup_solana.go` — Solana USDC deposit handler
- `service/billing.go` — Usage billing engine
- `service/pre_consume_quota.go` — Reserve cost before request
- `service/pricing.go` — Model pricing configuration

## Rules

- All JSON marshal/unmarshal through `common/json.go`
- Database code must work with SQLite, MySQL, and PostgreSQL
- Prefer GORM over raw SQL
- New channels: check if provider supports `StreamOptions`
- Optional scalar fields in relay request DTOs MUST use pointer types with `omitempty`

## Testing

Run: `go test ./...`

Prefer table-driven tests with explicit inputs and expected outputs. Use `github.com/stretchr/testify/require` and `assert`.

---
> Source: [Solqora/solqora-core](https://github.com/Solqora/solqora-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
