## tmpmail

> **Generated:** 2026-03-31 21:38:42 CST

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-31 21:38:42 CST  
**Commit:** 3afb500  
**Branch:** main

## Scope
- Applies repo-wide.
- Nearest child `AGENTS.md` overrides local work.
- Read next: `src/AGENTS.md`, `src/routes/AGENTS.md`, `frontend/AGENTS.md`, `frontend/app/AGENTS.md`, `frontend/lib/AGENTS.md`.

## Overview
TmpMail is a Rust Axum API plus a Next.js App Router frontend. Persistence is PostgreSQL-only; startup accepts either an explicit `TMPMAIL_DATABASE_URL` or the bundled compose `TMPMAIL_POSTGRES_*` settings, then applies `migrations/` automatically.

## Structure
```text
.
├── src/                 # Rust backend core, workers, auth, storage, routes
├── migrations/          # PostgreSQL schema
├── frontend/            # Next.js app, proxies, UI, i18n
├── scripts/             # Preferred local dev + smoke helpers
├── Dockerfile           # API image
├── frontend/Dockerfile  # Frontend image
├── compose.yaml         # Default orchestration for API + frontend + postgres + inbucket
├── .env.example         # Canonical env template
├── README.md            # Operational behavior and deployment notes
└── data/                # Runtime state only; ignored
```

## Where to look
| Task | Location | Notes |
|---|---|---|
| Bring the stack up/down | `compose.yaml`, `scripts/dev-up.sh`, `scripts/dev-down.sh` | Default workflow is plain `docker compose`; `dev-up` can auto-create `.env`, ask for script language, prompt for required values plus proxy mode, hard-fail when host `25/TCP` is not both published to Inbucket and reachable on `127.0.0.1:25`, wait for `api` and `frontend` readiness, print DNS follow-up, warn-but-continue when `TMPMAIL_MAIL_EXCHANGE_HOST` is not resolvable yet, hard-fail only when it resolves but `:25` cannot connect, and then optionally prompt to clean TmpMail-owned dangling Docker images |
| Backend startup | `src/main.rs`, `src/app.rs` | Config, worker spawning, router assembly |
| HTTP route map | `src/routes/mod.rs` | `api_router()` vs `stream_router()` |
| Storage backend | `src/app_store.rs`, `src/pg_store.rs` | PostgreSQL-only business storage |
| Runtime overrides | `src/config.rs`, `src/admin_state.rs` | Env parsing plus persisted admin overrides |
| Frontend shell | `frontend/app/[locale]/layout.tsx`, `frontend/app/[locale]/page.tsx` | Locale shell, providers, inbox/guest flow |
| Frontend proxy edges | `frontend/app/api/mail/route.ts`, `frontend/app/api/sse/route.ts` | REST proxy vs SSE proxy |
| Configurable admin path | `frontend/proxy.ts`, `frontend/lib/admin-entry.ts` | Never assume `/admin` is fixed |
| Frontend transport/helpers | `frontend/lib/api.ts` | Central fetch, errors, session-adjacent helpers |
| Minimal end-to-end validation | `scripts/smoke.sh` | Health, auth, inbox, raw message, frontend `/en` |

## Code map
| Symbol | Type | Location | Refs | Role |
|---|---|---|---:|---|
| `build_router` | fn | `src/app.rs` | 6 | Merges protected API routes with the separate stream router |
| `api_router` | fn | `src/routes/mod.rs` | 2 | Canonical REST route table |
| `AppStore` | struct | `src/app_store.rs` | 67 | PostgreSQL facade used across routes and workers |
| `AuthProvider` | component | `frontend/contexts/auth-context.tsx` | 4 | Frontend auth state boundary mounted from locale layout |

## Shared verification
- Backend changes: `cargo test` and `cargo build --release`
- Frontend changes: `cd frontend && npm run lint`
- Frontend route/env/proxy changes: also run `cd frontend && npm run build`
- Repo-level sanity: `./scripts/smoke.sh`
- Storage or schema changes: verify happy path plus migration/restore behavior

## Conventions
- Prefer plain `docker compose up -d --build` / `docker compose down` for the default deployment path; `scripts/dev-up.sh` and `scripts/dev-down.sh` are optional local helpers, and `dev-up` can auto-create `.env` from `.env.example`, ask for script language with Simplified Chinese as the default, prompt for missing required deployment values plus `TMPMAIL_TRUST_PROXY_HEADERS` deployment mode, validate that host `25/TCP` really publishes to Inbucket SMTP and is reachable on `127.0.0.1:25`, wait for `api` / `frontend` readiness, print a single mail-host A/AAAA DNS step after startup, warn-but-continue when `TMPMAIL_MAIL_EXCHANGE_HOST` is not resolvable yet, hard-fail only when it resolves but `:25` cannot connect, and then optionally prompt to clean TmpMail-owned dangling Docker images without touching other projects' caches.
- `.env.example` is the source of truth for new settings. Update `README.md` when admin, JWT, transport, or deployment knobs change.
- Env namespaces are deliberate: `TMPMAIL_*` backend/runtime plus Docker build overrides, `NEXT_PUBLIC_TMPMAIL_*` browser-visible frontend, and `INBUCKET_*` compose-internal container envs.
- Security-sensitive envs now include `TMPMAIL_JWT_SECRET`, `TMPMAIL_ALLOW_INSECURE_DEV_SECRETS`, `TMPMAIL_ADMIN_PASSWORD`, `TMPMAIL_ADMIN_PASSWORD_MODE`, `TMPMAIL_DATABASE_URL`, `TMPMAIL_POSTGRES_PASSWORD`, `TMPMAIL_TRUST_PROXY_HEADERS`, `TMPMAIL_CONTAINER_UID`, `TMPMAIL_CONTAINER_GID`, and `TMPMAIL_POSTGRES_BIND_IP`; keep docs and compose defaults aligned when they change.
- Main compose now wires TmpMail to the built-in `inbucket` service over the Docker network; do not document `TMPMAIL_INGEST_MODE` / `TMPMAIL_INBUCKET_BASE_URL` as required for the default same-host deployment path.
- Main compose now includes a one-shot `secrets-init` step that persists generated JWT / PostgreSQL secrets under repo-local `./data/runtime-secrets`; do not move that runtime state outside the project directory.
- Default compose persistence must stay under repo-local `./data/`; avoid introducing named volumes or host paths outside the project directory for runtime state.
- Default direct-compose deployments must treat proxy headers as untrusted; only enable `TMPMAIL_TRUST_PROXY_HEADERS` when a trusted reverse proxy is explicitly in front and overwrites forwarding headers.
- Public Prometheus scraping is opt-in through `TMPMAIL_PUBLIC_METRICS_ENABLED`; do not assume `/metrics` is always exposed.
- `/events` lives outside the protected API middleware path; use `TMPMAIL_SSE_CONNECTION_LIMIT` to cap long-lived SSE fanout.
- OTP outbound SMTP now supports `TMPMAIL_SMTP_SECURITY=plain|starttls|tls`; legacy `TMPMAIL_SMTP_STARTTLS` is compatibility-only and should not be the primary documented knob anymore.
- Localized frontend entrypoints are `/zh` and `/en`; the default locale is `zh`.
- Frontend page files stay thin. Heavy UI lives in `frontend/components/`; shared fetch/session/config logic lives in `frontend/lib/`.
- There is no dedicated frontend test harness; lint/build/smoke are the expected safety net.

## Collaboration
- Commit subjects follow the existing repo style: short, imperative Chinese.
- PRs should call out user-visible impact, config or migration changes, and manual verification steps.
- Include screenshots for UI/admin changes.
- When config surfaces change, mention `.env.example`, `compose.yaml`, `README.md`, and `migrations/` updates explicitly.

## Anti-patterns
- Do not add or revive a second frontend tree outside `frontend/`. The active frontend is `frontend/`.
- Do not commit `.env`, `data/`, or legacy generated `inbucket.compose.yml` / `inbucket.env` / `inbucket-data/` artifacts.
- Do not assume `cargo run` reads `.env`; the Rust binary reads process env directly.
- Do not assume `cargo run` or tests read `.env`; export `TMPMAIL_DATABASE_URL` explicitly outside Docker unless you are inside the bundled compose path, and remember that compose-specific `*_FILE` secret wiring does not exist in plain host runs.
- Do not assume env always wins over persisted admin-state overrides for mail/DNS/SMTP behavior.
- Do not hardcode `/admin`; `TMPMAIL_ADMIN_ENTRY_PATH` and `frontend/proxy.ts` can rewrite public admin paths.

## Commands
```bash
cp .env.example .env
./scripts/dev-up.sh
docker compose up -d --build
docker compose down
./scripts/smoke.sh
cargo test
cargo build --release
cd frontend && npm ci && npm run lint
cd frontend && npm run build
```

## Notes
- `frontend/Dockerfile` uses `npm ci --legacy-peer-deps`; local docs still use plain `npm ci`.
- API Docker builds accept `TMPMAIL_CARGO_*` compose args for Cargo protocol / mirror / retry tuning. In slow or filtered networks, prefer `TMPMAIL_CARGO_MIRROR=sparse+https://rsproxy.cn/index/`.
- Frontend Docker builds accept `TMPMAIL_NPM_REGISTRY` and `TMPMAIL_NPM_FETCH_TIMEOUT`; in slow or filtered networks, prefer `TMPMAIL_NPM_REGISTRY=https://registry.npmmirror.com`.
- API and frontend runtime containers are expected to run as non-root with hardened compose defaults; if bind-mounted `./data` needs permission alignment, use `TMPMAIL_CONTAINER_UID` / `TMPMAIL_CONTAINER_GID`.
- Linux Do OAuth secrets can be seeded from `TMPMAIL_LINUX_DO_CLIENT_SECRET`; runtime persistence lives in PostgreSQL.
- Default documented deployment path is plain `docker compose`; do not reintroduce a separate same-host Inbucket deployment script.
- Main compose exposes Inbucket SMTP on host `25/TCP`, keeps Inbucket Web/API on the internal Docker network, and uses dedicated publish-only bridge networks for host-facing Inbucket/PostgreSQL ports so `frontend` still stays isolated from `postgres` / `inbucket`.
- `TMPMAIL_MAIL_EXCHANGE_HOST` must resolve to a public DNS-only SMTP endpoint that really accepts `25/TCP`; `dev-up` now probes that endpoint after startup to catch false-success deployments early.

## Update policy
If boundaries, commands, env knobs, routing, or verification rules change, update the nearest `AGENTS.md` in the same PR.

---
> Source: [Ign1x/tmpmail](https://github.com/Ign1x/tmpmail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
