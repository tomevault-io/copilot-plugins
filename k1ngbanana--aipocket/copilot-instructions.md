## aipocket

> Guidelines for AI agents (Claude Code, Copilot, Cursor, etc.) working on this codebase.

# AGENTS.md

Guidelines for AI agents (Claude Code, Copilot, Cursor, etc.) working on this codebase.

## Project Overview

**AIPocket** scans for exposed AI infrastructure via FOFA + Shodan (+ GitHub artifact source), extracts and validates leaked API key/URL pairs, checks balances, and flags high-value findings.

Monorepo layout:
- `crates/` — Rust 2024 workspace；Axum + Clap + SQLx + redis-rs + Reqwest
- `frontend/` — React 19 + Vite + Tailwind v4 + shadcn/ui, `pnpm`
- Infra: PostgreSQL 16, Redis 7 (see `docker-compose.yml`)

## Before You Start

1. Read `CLAUDE.md` for architecture, commands, and conventions.
2. Read `.env.example` to understand configuration.
3. Never commit secrets or `.env` files.

## Backend Guidelines

- Source: `crates/aipocket-*`，依赖方向保持 `api → services → discovery/prober/clients/db/core`。
- Validate: `cargo test --workspace`
- Lint: `cargo fmt --all -- --check && cargo clippy --workspace --all-targets -- -D warnings`
- Coverage: `cargo llvm-cov --workspace --fail-under-lines 86`
- All HTTP I/O is async through shared `reqwest::Client`.
- Config env names/defaults must stay compatible with `.env.example`; never add a required env without a compatible default.
- Schema changes must be additive and idempotent. Never drop/rebuild production tables or change Redis key formats.
- New API routes belong in `aipocket-api`; business logic belongs in `aipocket-services`, not handlers.

## Frontend Guidelines

- Source: `frontend/src/`
- Build check: `cd frontend && pnpm build`
- Lint: `cd frontend && pnpm lint`
- UI components use shadcn/ui (`components/ui/`). Don't install alternative component libraries.
- Server state: `@tanstack/react-query`. Don't use Redux or other state managers.
- Routing: `react-router-dom` v6.
- API client: `lib/api.ts`. All backend calls go through this module.

## Common Tasks

| Task | Steps |
|------|-------|
| Add a platform prober | Implement `Prober` in `aipocket-prober`, register it, add behavioral tests |
| Add an API endpoint | Add route/DTO in `aipocket-api`, reuse a service/repository, add a contract test |
| Add a frontend page | Create page in `pages/`, add route in `App.tsx`, add sidebar link |
| Add an env var | Add a backward-compatible default to Rust `Settings` and `.env.example` |

## Do Not

- Introduce synchronous HTTP calls in Rust backend code
- Store runtime state, credentials, or scan results in git-tracked files
- Modify production PG/Redis volumes, URLs, database names, or ports for the Rust migration
- Add destructive migrations, flush Redis, or require `.env` edits for deployment
- Skip `cargo test --workspace`, fmt, clippy, and frontend build validation

---
> Source: [k1ngbanana/aipocket](https://github.com/k1ngbanana/aipocket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
