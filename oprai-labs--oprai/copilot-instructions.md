## oprai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OPRAI is a DeFi-native conversational AI assistant for Solana. It transforms natural language into on-chain actions (swaps, token launches, staking, transfers) with wallet-based authentication (SIWS).

## Common Commands

### Setup & Run (Polyglot — primary)
```bash
cp .env.example .env              # Pre-filled with generated secrets; add your own OpenAI key
make install                      # Install all deps (Node.js + Go + Rust + Python)
make proto                        # Generate gRPC stubs from proto/ (Go, Python, Rust)
make build-python                 # Create Python venvs and install deps
make dev-infra                    # Start Postgres (5433), Redis (6379), Qdrant (6333) in Docker
make migrate                      # Run database migrations
make dev-all                      # Start infra + all 7 polyglot services via honcho
make health                       # Check gateway aggregated health
```

### Individual Service Groups
```bash
make dev-go                       # Gateway + Auth + Admin (Go)
make dev-rust                     # Solana service (Rust)
make dev-python                   # Chat + Memory (Python, uvicorn --reload)
make dev-angular                  # Angular frontend on :3000
```

### Build
```bash
make build-go                     # Compile all Go services
make build-rust                   # cargo build --release for Rust
make build-angular                # ng build --configuration production
make build-all                    # proto + all services + frontend
```

### Legacy Node.js Stack (still compiles, gradually being replaced)
```bash
pnpm install && pnpm build        # Build all legacy packages/services via Turborepo
pnpm dev                          # Start legacy services (excludes oprai-web)
pnpm test                         # Run all Vitest tests (legacy)
pnpm --filter @oprai/auth-service dev        # Single legacy service
pnpm --filter @oprai/auth-middleware test     # Single legacy package test
pnpm --filter @oprai/chat-service db:migrate # Single service migrations
pnpm dev:admin                    # Legacy admin-service + admin-panel only
```

### Database
```bash
make migrate                      # All service migrations
make backup                       # pg_dump to backups/
make restore                      # Restore from latest (or BACKUP=path)
make reset                        # Drop + recreate (creates backup first, requires confirmation)
```

### Docker (Full Stack)
```bash
make docker-up                    # Build + start full polyglot stack with monitoring
make docker-down                  # Stop
make docker-logs                  # Tail logs
docker compose up -d              # Legacy Node.js stack only
```

## Architecture

**Dual-stack monorepo**: polyglot services (primary) coexist with legacy Node.js services during migration. Inter-service communication is **gRPC + Protobuf** (15 proto files under `proto/`). Monitoring via **Prometheus** (:9090) + **Grafana** (:3333).

```
Frontend (Angular :3000) → Bearer JWT → Gateway (Go :3001)
                                          ├── auth-service (Go :3010/50051)    → auth_schema
                                          ├── chat-service (Python :3020/50052) → chat_schema
                                          ├── solana-service (Rust :3030/50053) → solana_schema
                                          └── memory-service (Python :3040/50054) → memory_schema

Admin Panel (Angular :3000) → Bearer JWT → admin-service (Go :3050/50055) → cross-schema SQL
```

### Polyglot Services (new, primary)
| Service | Path | Language | Framework | Port (HTTP/gRPC) |
|---------|------|----------|-----------|------------------|
| Gateway | `services/gateway-go/` | Go | Chi, gobreaker | 3001 |
| Auth | `services/auth-service-go/` | Go | Chi, pgx, golang-jwt, go-redis | 3010 / 50051 |
| Chat | `services/chat-service-py/` | Python | FastAPI, LangChain, SQLAlchemy | 3020 / 50052 |
| Solana | `services/solana-service-rs/` | Rust | Actix-Web, Tonic, solana-sdk, Diesel | 3030 / 50053 |
| Memory | `services/memory-service-py/` | Python | FastAPI, qdrant-client, OpenAI | 3040 / 50054 |
| Admin | `services/admin-service-go/` | Go | Chi, sqlc, bcrypt | 3050 / 50055 |
| Frontend | `apps/oprai/` | TypeScript | Angular 19, standalone components | 3000 |
| OpraiOS | `opraios/` | Python | Pydantic, OpenAI, solana-py | standalone |

### Additional Services (newer, not yet in the table above)
- **`services/solana-service-ts/`** — TypeScript rewrite of the Rust solana-service using official protocol SDKs (Jupiter, Kamino, Drift, Marinade, Meteora, MarginFi). Runs alongside the Rust service: `make dev-solana-ts` (PORT 3030) or via Procfile as `solana-ts` (PORT 3031). The two are interchangeable backends for the same `/actions/*` surface — the Rust service is still the documented default; the TS one is the migration target.
- **`services/knowledge-ingestion-service/`** (Python, FastAPI, :3070) — admin REST API + scheduler (APScheduler) that crawls/chunks docs into the knowledge base. Sources configured via `source_configs/*.yaml` or at runtime. Has its own venv + Alembic migrations; created by `make build-python`, run in Procfile as `knowledge`.
- **`services/defi-query-service/`** (Python, FastAPI + CLI, :3150 — see memory) — standalone LLM orchestrator: `POST /query {question, jwt_token?} → {html, plain, tools_called}`. Wraps the gateway's DeFi GET endpoints as tools. Runnable as a CLI: `python3 main.py "What's the Jito tip floor?"`.

### Legacy Services (Node.js, being replaced)
`services/gateway/`, `services/auth-service/`, `services/chat-service/`, `services/solana-service/`, `services/memory-service/` — Express + Sequelize + Turborepo. (The Next.js `apps/chat-web/` and `apps/admin-panel/` frontends have been removed — `apps/oprai/` is now the only frontend; admin is a route within it.)

### Agent Platform (`agent-platform/`)
A separate, largely self-contained sub-project (own `README.md`, `Makefile`, `docker-compose.yml`, `services/`, `frontend/`, `programs/`, `proto/`, `migrations/`) — a blockchain-native platform to create, deploy, and monetize AI agents on Solana (NFT identity, marketplace, visual builder, multi-platform connectors). Distinct from both the polyglot services and `opraios/`. Its migrations are applied by the root `make migrate` (`agent-platform/migrations/00*.sql`). Treat it as its own project: read `agent-platform/README.md` and `agent-platform/Makefile` before working there.

### Shared Packages (legacy, used by legacy services)
- **`@oprai/types`** — TypeScript interfaces (auth, chat, solana, memory)
- **`@oprai/auth-middleware`** — Express JWT middleware
- **`@oprai/db-config`** — Sequelize connection factory + umzug migration runner
- **`@oprai/solana-common`** — Token registry (COMMON_TOKENS), Solana RPC helpers

### Gateway
Single entry point. JWT validation, `X-User-Wallet` + `X-Internal-Api-Key` header injection, rate limiting (100/min global, 20/min auth), health aggregation, circuit breaker (gobreaker).

### Auth Flow
```
1. POST /auth/nonce → { nonce, nonceId } (stored in Redis, 10-min TTL)
2. Client signs nonce with wallet (tweetnacl ed25519)
3. POST /auth/verify → { token, expiresAt } + sets HttpOnly cookie
4. Frontend keeps the JWT in MEMORY ONLY (not localStorage); the HttpOnly
   cookie is the durable copy and is sent automatically with every request
   via `credentials: 'include'`. On page reload, GET /auth/session restores
   the user from the cookie.
5. Gateway validates JWT → injects X-User-Wallet + X-Internal-Api-Key → proxies to service
```

**Wallet switching is security-sensitive**: `WalletService.accountChanged$` fires on every connect / disconnect / native account-change. `AppComponent` handler MUST call `authService.logout()` → `router.navigateByUrl('/')` → `authenticate()` in that order. Skipping the navigate leaves the previous wallet's chat-session URL active and the chat-shell keeps rendering its messages until the user opens a different conversation. `SessionStorageService` is namespaced per-wallet (`oprai-sessions:${wallet}`) — the in-memory state is cleared on logout, but the previous wallet's data stays safely on disk under its own key for next sign-in.

### Solana Action Flow
1. User sends natural language → chat-service → LLM with SOLANA_ACTION_PROMPT
2. LLM returns action blocks: `[ACTION:transfer] to=HwM... amount=1 token=SOL`
3. Frontend parses action blocks (intent-parser)
4. Frontend calls `/actions/quote` → `/actions/build` → user signs with wallet → submit TX

### Chat-service LLM configuration
- Provider toggle: `OPRAI_LLM_PROVIDER=openai` (default) | `anthropic`
- OpenAI responder: `gpt-5.4-mini` (Responses API), fallback `gpt-4o-mini` (Chat Completions). Tool-call leakage from Harmony channels is filtered in `_strip_tool_call_leakage`. Bumped from nano on 2026-05-17 — nano hedged on ambiguous turns and lost tool-chain context.
- Anthropic responder: `claude-sonnet-5` (bumped from Haiku 4.5 on 2026-07-21 for far better prose/world-knowledge — Haiku answers read like literal English→Turkish translations). Thinking is disabled per-call for the chat responder (`_anthropic_thinking_kwargs` in `services/llm.py`) since Sonnet 5 defaults adaptive thinking ON. Prompt caching is wired (`cache_control: ephemeral` on the system block); repeat turns inside the 5-min TTL drop input cost ~90%.
- Intent classifier (`gpt-5.4-nano`) runs before the main responder to narrow the tool list — see `services/intent_router.py`.

### Chat-service tool registry — 4-place rule
Adding a `query_onchain` tool requires updates in ALL of these or the LLM will silently fail to find/use it:
1. `app/clients/market_data.py` — implementation (`async def`) + register in `_DISPATCH` (params: required, optional)
2. `app/services/action_schemas.py` — add to `QueryType` enum
3. `app/services/tool_selector.py` — add intent group(s) (`{portfolio, analysis, dex, …}`)
4. `app/prompts/solana_action_market_data.txt` (or matching domain prompt) — table row + parameter reference + at least one example

### Frontend token registry — single source of truth
`apps/oprai/src/app/core/services/market/tokens.generated.ts` is the only place mint addresses for well-known tokens should live. It's CI-validated by `scripts/verify-tokens.mjs` (catches typos and vanity-prefix collisions before they ship). `TokenRegistryService` exposes:
- `getToken(mint)` / `getBySymbol(symbol)` — lookups
- `ensureLoaded()` — fetches Jupiter strict list and merges
- `isStable(addressOrSymbol)` — combined Jupiter-tag + symbol/name heuristic; never reintroduce hardcoded `Set<string>` of stables.

### Database
- Single PostgreSQL instance (:5433), per-service schema isolation
- Schemas: `auth_schema`, `chat_schema`, `solana_schema`, `memory_schema`, `admin_schema`
- No cross-service foreign keys; services reference each other by string IDs
- Legacy: Sequelize ORM + umzug migrations
- Polyglot: pgx (Go), SQLAlchemy (Python), Diesel (Rust)

### Admin Service
Separate auth (username/password + bcrypt, `admin_schema.admin_users`). Cross-schema raw SQL. Does NOT go through gateway. Default admin: `admin`/`admin123`.

## Port Reference

| Service | HTTP | gRPC | | Infrastructure | Port |
|---------|------|------|-|----------------|------|
| Frontend | 3000 | — | | PostgreSQL | 5433 |
| Gateway | 3001 | — | | Redis | 6379 |
| Auth | 3010 | 50051 | | Qdrant HTTP | 6333 |
| Chat | 3020 | 50052 | | Qdrant gRPC | 6334 |
| Solana | 3030 | 50053 | | Prometheus | 9090 |
| Memory | 3040 | 50054 | | Grafana | 3333 |
| Admin | 3050 | 50055 | | | |
| Solana (TS) | 3030/3031 | — | | | |
| Knowledge | 3070 | — | | | |
| Marketing | 3100 | — | | | |
| DeFi Query | 3150 | — | (standalone) | | |
| OpraiOS MCP | 8000 | — | (standalone, optional) | | |

## Frontend Routes (Angular)

Main pages under `apps/oprai/src/app/features/`:
- **Chat** (`/`) — Home page, AI chat interface
- **Portfolio** (`/portfolio`) — Wallet holdings, token balances
- **Agents** (`/agents`) — AI agent management
- **Voice** (`/voice`) — Voice-based interactions
- **Admin** (`/admin`) — Admin panel (separate layout, bypasses gateway)

Legacy redirects: `/market`, `/explore`, `/trade`, `/settings`, `/tokens`, `/nft`, `/defi` → all redirect to `/`

## Protobuf & gRPC

15 proto files under `proto/` organized by domain: `common/`, `auth/`, `chat/`, `solana/`, `memory/`, `admin/`. Run `make proto` (or `./scripts/build-protos.sh [go|python|rust]`) to generate stubs:
- Go: `protoc-gen-go` + `protoc-gen-go-grpc` → `services/<svc>/proto/gen/go/`
- Python: `grpcio-tools` → `services/<svc>/proto_gen/`
- Rust: `tonic-build` via `build.rs` (generated at `cargo build` time)

## Environment Variables

Single `.env.example` at repo root (pre-filled with generated secrets). Key variables:

| Variable | Required | Description |
|----------|----------|-------------|
| `OPRAI_JWT_SECRET` | Yes | JWT signing secret |
| `OPRAI_INTERNAL_API_KEY` | Yes | Gateway-to-service shared key |
| `OPRAI_OPENAI_API_KEY` | Yes | OpenAI API key for LLM and embeddings |
| `DATABASE_URL` | Auto | Composed from `DB_SUPERUSER`/`DB_SUPERPASS`/`DB_SUPERDB` |
| `REDIS_URL` | No | Default: `redis://localhost:6379` |
| `QDRANT_URL` | No | Default: `http://localhost:6333` |
| `SOLANA_RPC` | No | Default: mainnet public |
| `OPRAI_ADMIN_JWT_SECRET` | Yes | Admin panel JWT secret |
| `BIRDEYE_API_KEY` | No | Market data API (proxied via gateway) |
| `JUPITER_API_KEY` | No | Jupiter API key |

## CI/CD

5 GitHub Actions workflows with path-based triggers:

| Workflow | Trigger paths | Steps |
|----------|---------------|-------|
| `go-services.yml` | `services/*-go/**` | vet, test, build, Docker push |
| `python-services.yml` | `services/*-py/**` | ruff, pytest, Docker push |
| `rust-service.yml` | `services/*-rs/**` | fmt, clippy, test, build, Docker push |
| `angular-frontend.yml` | `apps/oprai/**` | lint, test, build, Docker push |
| `proto-check.yml` | `proto/**` | buf lint, breaking change detection |

## Build Notes

### Legacy TypeScript
- `NodeNext` module resolution — all imports need `.js` extension (e.g., `import { env } from "./config/env.js"`)
- **TS2742 "inferred type cannot be named"**: Annotate `const router: Router = Router()` explicitly
- `@solana/spl-token` is ESM-only: use dynamic `import()` in CJS/NodeNext modules
- `fetch().json()` returns `unknown` in strict TS: cast with `as any`
- Test files excluded from `tsc` build via `tsconfig.json` (`"exclude": ["src/__tests__"]`); Vitest handles them separately
- Changing `@oprai/types` or `@oprai/auth-middleware` triggers rebuild of all dependent services

### Polyglot
- **Go**: Entry points at `cmd/<service>/main.go`. Run `go mod tidy` if build fails.
- **Rust**: Single-binary crate, entry at `services/solana-service-rs/src/main.rs` (no `cmd/` subdir). First build downloads crates (3-5 min). Proto stubs generated at build time via `build.rs` + tonic-build.
- **Python**: Services use venvs (`.venv/`). Install with `make build-python`. Run with `.venv/bin/uvicorn`.
- **Angular**: Angular 19 with standalone components, lazy-loaded modules. Build with `npx ng build` from `apps/oprai/`.
- **Proto generation**: Requires `protoc` (`brew install protobuf`), `protoc-gen-go`/`protoc-gen-go-grpc` (Go), `grpcio-tools` (Python).

### Process Manager
`Procfile.dev` defines the dev services for `honcho`: gateway, auth, admin, chat, memory, solana (Rust), solana-ts (:3031), knowledge (:3070), frontend. `make dev-all` starts infrastructure + all of them in one terminal with color-coded logs. The `knowledge` entry self-skips if its `.venv` is missing (run `make build-python` to enable).

### Frontend Design System
- **CSS Variables**: All styling uses `--op-*` prefixed tokens (e.g., `--op-bg-surface-1`, `--op-text-primary`, `--op-brand`)
- **Brand Colors**: Indigo (`#5b5fc7`) → Cyan (`#06B6D4`) gradient
- **Location**: `apps/oprai/src/styles.scss`
- **Note**: Admin pages use a separate token system (`--bg-primary`, `--text-primary`, etc.) — not migrated

## OpraiOS (Agent Platform)

Python package at `opraios/` for building, training, and deploying AI agents for Solana DeFi. Separate from polyglot services — has its own venv and dependencies.

### Setup & Run
```bash
cd opraios
python3 -m venv .venv && .venv/bin/pip install -e ".[dev]"
.venv/bin/pytest                    # Run tests
.venv/bin/python -m opraios.mcp.server  # Start MCP server
```

### Architecture
```
opraios/
├── core/               # Core framework (agent_builder, character, plugin_system)
├── plugins/            # DeFi protocol plugins (jupiter, orca, kamino, drift, etc.)
├── templates/          # 16 agent templates (trading, yield, security)
├── mcp/                # Claude Code MCP integration
├── runner/             # Strategy runner daemon + scheduler
└── tests/              # pytest test suite (25+ test files)
```

### Key Components
- **AgentBuilder** — Fluent API for agent creation (`core/agent_builder.py`)
- **Plugin System** — Actions, Providers, Evaluators for DeFi protocols
- **Character System** — JSON-based agent personalities
- **Strategy Runner** — Daemon-based job scheduler with cron support
- **Safety System** — Fund movement protection with wallet limits
- **Cost Tracker** — LLM API cost monitoring

### Testing
```bash
.venv/bin/pytest tests/                    # All tests
.venv/bin/pytest tests/test_simulation.py  # Single file
.venv/bin/pytest -k "gas"                  # Filter by name
```

### Plugin Development
Plugins live in `opraios/plugins/`. Each plugin has:
- `plugin.json` or `manifest.json` — metadata
- `plugin.py` or `main.py` — entry point
- Actions, Providers, Evaluators as needed

Install plugins via `PluginManager.install(source)` — supports local paths, GitHub repos, and zip URLs.

---
> Source: [Oprai-Labs/oprai](https://github.com/Oprai-Labs/oprai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
