## asgard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Asgard — a single-binary distributed job management platform that unifies **resident processes (app)**, **cron-like scheduled jobs (job)**, and **one-shot timed jobs (timing)**. One Go binary dispatches via cobra to subcommands `web` / `master` / `agent` / `guard` / `cron` / `debug`. Three node roles cooperate over gRPC: **web** is the control plane, **master** is the registry/state sink, **agent** is the worker that actually spawns OS processes.

## Current state — read this first

The project is **mid-migration** (TASKS Phase 0~5, see `doc/TASKS.md`):

- **Phase 0** (planning) is complete — Vue 3 + Vite + TS + Element Plus + Pinia + axios + SSE chosen; `/api/v1` is the API prefix; auth is dual-track `Authorization: Bearer <jwt>` + legacy DES cookie.
- **Phase 1** (backend API) is in progress — `/api/v1` group is registered (`web/router.go` + `web/routers/api_router.go`); 11 resource groups + SSE handlers are wired; old HTML controllers have been moved to `web/legacy/` and `doc/legacy-templates/`, which carry `//go:build ignore` and are **not for new code**.
- **Phase 2-5** (frontend migration → cutover) are still in flight.

**Rule:** Before any non-trivial change, scan `doc/TASKS.md` §3 to avoid stepping on a sibling task. Start → mark `[/]`, finish/blocked → mark `[x]/[-]` and append a line to §4.

## Build, run, test

```sh
# Build (single binary, CGO off; uses Go 1.16 toolchain — see "known traps" below)
go build -o Asgard .

# Run a node (each needs a conf/app.yaml — see README §部署 for templates)
./Asgard web    -c conf    # JSON API on :12345 (no HTML since cutover)
./Asgard master -c conf    # gRPC on :9527
./Asgard agent  -c conf    # gRPC on :27149
./Asgard guard  -c conf [-s runtime/asgard_guard]   # supervisor-style, Unix socket
./Asgard cron   -c conf [-s runtime/asgard_cron]   # crontab-style, Unix socket
./Asgard debug ...                                    # mail / RPC tools
./Asgard agent status [-p 27149]                      # inspect local agent

# Tests (Go) — there is only one in-tree test file today:
go test ./web/utils/ -run TestIssueAndParseToken -v
go test ./web/utils/ -run TestParseToken -v
# (no project-wide test runner or coverage gate exists; add new tests per-package)

# Frontend (web-admin/)
cd web-admin
npm install
npm run build          # outputs web-admin/dist/, packaged by .goreleaser.yml
npm run test           # Vitest unit
npm run e2e            # Playwright (config: playwright.config.ts)
npm run dev            # Vite dev server; /api/* proxied to VITE_BACKEND_TARGET (default :12345)

# Regenerate gRPC stubs (requires legacy protoc-gen-go v1.x; new toolchains need adjustment)
bash scripts/protoc.sh

# Snapshot release (multi-platform: darwin/linux × i386/amd64/arm/arm64)
goreleaser release --snapshot --clean
```

There is **no** `Makefile`, `golangci-lint` config, or Go test harness in the repo — don't invent one before checking with the user.

## High-level architecture

```
                ┌────────────┐   gRPC    ┌──────────┐   gRPC   ┌────────┐
   Browser  ──▶ │  Asgard    │ ────────▶ │  master  │ ───────▶ │ agent  │──▶ os/exec
  (web-admin/   │  web       │           │ (registry│          │(managers
   SPA + SSE)   │  (Gin API) │ ◀───────  │ + state) │ ◀──────  │ + runtimes)
                └─────┬──────┘  gRPC/JOB └────┬─────┘  archive/ └────────┘
                      │                      │        monitor/
                      ▼                      ▼        exception
                ┌────────────┐         ┌──────────────────┐
                │  MySQL     │         │  Redis (cache)   │
                │ (incl.     │         │  (users/groups)  │
                │  monitors_ │         └──────────────────┘
                │  YYYYMM)   │
                └────────────┘
```

- **`web/`** — Gin server, JSON-only after cutover. `web/router.go` mounts `routers.SetupAPIRouter` under `/api/v1`; the rest of the tree lives in `web/middlewares/` (CORS + `APIAuth` + `APIAuthAdmin`) and `web/controllers/api_*.go` (one file per resource). Response helpers in `web/utils/respose.go`; SSE handlers in `api_sse.go`.
- **`cmds/<sub>/`** — cobra entry per subcommand. Each follows the same recipe: viper-load `conf/app.yaml` → register avenger components (`db/cache/logger/mail`) → init the relevant manager/controllers → `runtimes.Wait(...)` for SIGTERM/SIGINT.
- **`models/` + `services/` + `providers/`** — GORM persistence, business logic, and a service-singleton container (wires the 11 services in `providers/service.go` `init()`). All `WhereAndOrder / PageListbyWhereString` queries are **string-concatenated** (see "known traps").
- **`server/` + `clients/`** — gRPC server implementations and client wrappers. `clients/base.go` provides a Unix-socket dialer for `guard`/`cron`.
- **`managers/` + `runtimes/`** — agent-side lifecycle. `runtimes/cmd.go` defines the `Command` abstraction embedded by `App/Job/Timing`; `runtimes/monitor.go` holds the CPU/mem sampler; archive/exception types live alongside.
- **`registry/`** — etcd integration for master HA (leader election, `MASTER_CLUSTER_TTL=10s` lease) and the round-robin resolver agent clients use in cluster mode.
- **`protos/` + `rpc/`** — proto source and generated stubs. **All model↔protobuf conversion is centralized in `rpc/common.go`** (`Format/Build/Parse` + `BuildXConfig`) — never re-map in controllers/managers/services.
- **`constants/`** — the source of truth for status int64s, error codes, defaults; `viper` overrides at runtime. Any status value change must also be reflected in `models/*` defaults, `services/*` transitions, and `web/utils/format.go` `GetObjectName`.

## Conventions worth knowing

- **Module:** `module Asgard` (no internal modules); packages named after directory, singular, no underscores. Imports are full-path `Asgard/<pkg>`.
- **Auth:** JWT HS256 via `web.jwt_secret` (TTL `web.jwt_ttl`, default 7200s) **or** legacy DES cookie (`web.cookie_salt`, exactly 8 bytes — DES limit; wrong length blows up at first login, not at startup). `APIAuth` middleware accepts both.
- **State values** are `int64`, centralized in `constants/constant.go`. Common sets: `AGENT_*` (ONLINE=1/OFFLINE=0/FORBIDDEN=-1), `APP/JOB_STATUS_*` (RUNNING=1/PAUSE=2/STOP=0/UNKNOWN=-2/DELETED=-1), `TIMING_STATUS_*` adds FINISHED=3. `TYPE_*` (1=agent … 6=user) feeds the operation/exception tables.
- **Realtime:** SSE only (T-010). Events: `log` (streaming lines) and `point` (monitor samples) plus `ping`. `interval` query controls push frequency.
- **Cluster:** only when `master.cluster: true` / `agent.master.cluster: true`. Agents use `grpc.WithDefaultServiceConfig({loadBalancingPolicy:"round_robin"})` via `registry.NewResolver`; master non-leader skips `MoniterMaster` to avoid double-probing.
- **Comment density:** constant keys, SQL comments, and `format.go` display names are largely in Chinese — keep them that way, do not retro-translate.
- **Bilingual UI:** response bodies and the new `web-admin/` are English/Vue-driven; legacy templates (in `doc/legacy-templates/`) are Chinese — both are intentional.

## Known traps (don't get bitten)

- **Go toolchain:** `go.mod` pins `go 1.16` and `gopsutil v2.19.11`, which **fails to build on new macOS SDKs** (`KinfoProc` gone in `process_darwin.go`). For local macOS work either use a Go 1.16~1.20 toolchain or bump `gopsutil` to v3+.
- **`protoc` script:** `scripts/protoc.sh` uses the legacy `plugins=grpc` syntax, which only `protoc-gen-go v1.x` understands. Modern toolchains need `protoc-gen-go-grpc` and adjusted flags.
- **Stale agent client cache:** `providers/client.go` caches gRPC connections by `agent.ID` **forever**. If an agent changes IP/port, you must restart master (or clear the cache) — there is no TTL.
- **Parameterless SQL:** `services/*` builds `Where(...)` with `fmt.Sprintf("%v", userInput)` style — no real parameterization. New endpoints that filter on user-controlled fields are injection-prone.
- **No test gate:** there is one Go test file (`web/utils/jwt_test.go`) and `web-admin/` has Vitest + Playwright, but no CI enforcement of either — adding tests is encouraged but don't claim "the build is green" by running nothing.
- **Don't import `web/legacy/`** in new code; it's gated by `//go:build ignore` and exists for historical reference only.
- **viper-panics-on-missing-config:** missing `conf/app.yaml` panics at startup. `conf/` is gitignored, so deployments must ship it explicitly.
- **`cookie_salt` length** isn't validated at boot — set it to 8 bytes or login will fail at first request, not at startup.

## Pointers

- Architecture & node responsibilities: `README.md` + `doc/Asgard.png`
- Full developer guide (directory map, gRPC protocol, lifecycle, recipes): `AGENTS.md`
- Active task list and phase state: `doc/TASKS.md` (single source of truth — update §4 on every status change)
- JSON API contract: `doc/API.md`
- Performance baseline (placeholder, T-502): `doc/PERFORMANCE.md`
- Frontend project: `web-admin/README.md`

---
> Source: [dalonghahaha/Asgard](https://github.com/dalonghahaha/Asgard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
