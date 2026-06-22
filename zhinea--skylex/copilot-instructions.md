## skylex

> Skylex is a self-hosted database control plane. The Go backend has two binaries (`skylex-server` and `skylex-agent`) that talk over gRPC/Connect-RPC, plus a Vite + React Router 7 UI in `ui/`.

# Skylex — Agent Notes

Skylex is a self-hosted database control plane. The Go backend has two binaries (`skylex-server` and `skylex-agent`) that talk over gRPC/Connect-RPC, plus a Vite + React Router 7 UI in `ui/`.


## Quick start

```bash
# Backend — build both binaries and run the server locally
make build
make dev                                    # starts etcd + minio via docker compose, then runs cmd/server with config.example.yaml
make dev-server                             # runs cmd/server with config.example.yaml (no extra services)

# Full reference stack in Docker Compose
make docker-up                              # builds images and starts server + 3 agents + etcd + minio
make docker-down
```

## Project layout

- `cmd/server` → `skylex-server` binary (control plane)
- `cmd/agent` → `skylex-agent` binary (runs on each DB node)
- `cmd/cli` → empty placeholder for a future `skylexctl`
- `internal/server`, `internal/agent`, `internal/backup`, `internal/db`, `internal/postgres`, `internal/dcs` → internal packages
- `pkg/` → empty; public packages go here if needed
- `proto/skylex/v1/` → protobuf service definitions
- `gen/` → generated Go code from `buf generate` (do not hand-edit)
- `ui/` → Vite + React Router 7 + Tailwind CSS v4 frontend
- `deploy/docker-compose/` → reference deployment and Dockerfiles

## Developer commands

| What | Command |
|------|---------|
| Build both binaries | `make build` |
| Build server only | `make build-server` |
| Build agent only | `make build-agent` |
| Run server locally with dev deps | `make dev` |
| Run server locally (no extra services) | `make run-server ARGS=path/to/config.yaml` or `make dev-server` |
| Run agent locally | `make run-agent ARGS='--server localhost:9090 --token dev-token'` |
| Run all Go tests | `make test` |
| Lint Go code | `make lint` (uses `golangci-lint`; no repo-level config file) |
| Regenerate protobuf | `make proto` (runs `buf lint && buf generate`) |
| Clean build artifacts | `make clean` |
| UI dev server | `cd ui && npm run dev` → `http://localhost:5173` |
| UI typecheck | `cd ui && npm run typecheck` |
| UI production build | `cd ui && npm run build` |

## Configuration

- Server config is YAML; pass the path as the first argument (`./skylex-server config.yaml`). `make dev` uses `config.example.yaml`.
- Settings are merged with `koanf`: YAML file + env vars. Env vars use the prefix `SKYLEX_` and nested keys become `_` (e.g. `SKYLEX_DATABASE_DSN`, `SKYLEX_AUTH_JWT_SECRET`).
- `config.example.yaml` is committed and works as-is for local development.
- Defaults exist for most values; see `internal/server/config.go`. `auth.jwt_secret` defaults to `change-me-in-production` in `config.example.yaml` so dev sessions survive restarts. If `auth.jwt_secret` is left empty, a random secret is generated on startup and a warning is logged; existing JWTs will not validate after a restart.
- Agent settings are layered: CLI flags take precedence over environment variables, which take precedence over the config file (`/etc/skylex/agent.yaml` by default), with `internal/agent/config.go` defaults as the fallback.

```bash
# With flags
make run-agent ARGS='--server localhost:9090 --token dev-token'

# With a token file (recommended for production to avoid leaking secrets in shell history)
make run-agent ARGS='--server localhost:9090 --token-file /etc/skylex/token'
```

- Environment variables are still supported for backwards compatibility and container deployments (`SKYLEX_AGENT_TOKEN`, `SKYLEX_SERVER_ADDR`, `SKYLEX_HOSTNAME`, `SKYLEX_PORT`, `SKYLEX_PG_DATA_DIR`, and `SKYLEX_AGENT_CONFIG`).

## Cluster provisioning workflow

### Node selection
- Cluster creation requires explicit node selection via `node_ids` in `CreateClusterRequest`.
- The first `node_id` becomes the **primary**; all remaining IDs become **replicas**.
- All selected nodes must be unassigned (`cluster_id` empty) and have a linked agent (`agent_id` non-empty).
- `GetByIDs` validates that every supplied ID exists and returns an error listing any missing nodes.

### Service location
- Each cluster has a `service_location` (stored on both the `clusters` and `nodes` tables):
  - `native` — PostgreSQL runs directly on the agent host.
  - `docker` — PostgreSQL runs inside the official `postgres:<version>` container named `skylex-postgres`, with the agent data directory mounted as a persistent volume.
- Agents report `docker_available` at registration; the server logs a warning when Docker is requested but unavailable on a node (it does not block creation).

### Provisioning command sequence
For **native** nodes the server queues only `pg_preflight` initially. The agent reports back:
- `NOTHING_FOUND` → safe to install; server queues `pg_install_native` (if not already installed) then the standard role commands.
- `PG_EXISTS` (existing PostgreSQL or data directory) → node transitions to `installation_state=conflict`; the UI surfaces a per-node conflict card with three choices:
  - **Adopt** — queue `pg_adopt_native` then role init commands (no data loss).
  - **Purge & Reinstall** — queue `pg_purge_native` → `pg_install_native` → role init commands (data loss, requires explicit user confirmation).
  - **Abort** — mark cluster `FAILED`, no further provisioning.

For **Docker** nodes the server skips preflight and immediately queues `pg_install_docker` → `pg_init` → `pg_start` → replication commands.

After installation, the standard init sequence is:
- Primary: `pg_init`, `pg_start`, `pg_create_repl_user`
- Replica: `pg_basebackup`, `repoint_replica`, `pg_start`

### Installation states (`nodes.installation_state`)
| State | Meaning |
|-------|---------|
| `pending_preflight` | Awaiting `pg_preflight` result |
| `nothing_found` | Preflight passed; clean install can proceed |
| `conflict` | Existing PostgreSQL or data found; user action required |
| `installing` | Install/purge in progress |
| `installed` | PostgreSQL successfully installed and running |
| `adopted` | Existing installation adopted without reinstall |
| `failed` | Install or conflict resolution failed |

### Conflict resolution RPC
`NodeService.ResolveInstallationConflict` accepts `node_id` + `action` (`ADOPT`, `PURGE`, `ABORT`). Only nodes in `installation_state=conflict` assigned to a cluster in `CREATING` status are eligible.

### Native installation
- Detects `apt-get`, `dnf`, `apk`, or `zypper` and installs `postgresql-<version>` packages.
- All `exec.Command` calls use argument slices (no shell interpolation); user input is never passed to a shell string.
- The agent process must have sufficient OS package privileges for native installs.

### Docker provisioning
- Docker Engine must already be installed and reachable by the agent user. Skylex does not install Docker Engine.
- The container is named `skylex-postgres` and is started with `--restart unless-stopped`.

### Observability
- Cluster detail pages show per-node `installation_state` badges and a live command-log tail via the existing `useCommandLogs` hook.
- All sensitive values in command output are redacted by `RedactSecrets` before being stored or streamed to the server.

## Database and migrations

- Server uses embedded SQLite by default via `modernc.org/sqlite`.
- Migrations are embedded in `internal/db/migrations/*.sql` and applied automatically when the server starts. The migration table is `schema_migrations` and versioning is based on the first 14 characters of the filename.
- SQLite connection is intentionally limited to `SetMaxOpenConns(1)`.

## Protocol buffers and generated code

- Buf v2 is configured in `buf.yaml` (modules under `proto/`, lint + breaking rules) and `buf.gen.yaml` (generates Go gRPC + Connect-RPC into `gen/`).
- **Always run `make proto` after changing `.proto` files.** Never manually edit files under `gen/`.
- `go_package_prefix` is `github.com/zhinea/skylex/gen`.

## Testing

- `make test` runs `go test ./...`. Test packages: `internal/server`, `internal/db`, `internal/agent/installer`.
- Test coverage includes: cluster provisioning validation (`validateClusterSetting`, `installCommands`), installer logic (`PreflightResult`, `DockerCommandArgs`, `formatCommand`), migration sequential-numbering and idempotency, and cluster settings repository operations.
- The project has no CI workflows, no pre-commit hooks, and no `golangci-lint` config file yet.

## Docker Compose reference stack

- `make docker-up` starts: `skylex-server`, three `skylex-agent` instances, etcd, and MinIO.
- Server exposes `8080` (HTTP), `9090` (gRPC), and `9091` (metrics).
- MinIO console is on `9001`; S3 API on `9000`.
- Agent containers mount dedicated PostgreSQL data volumes (`pg-data-1`, etc.).
- Requires `docker compose` v2 and the env vars `SKYLEX_JWT_SECRET` / `SKYLEX_AGENT_TOKEN` only if you want to override their defaults (`change-me-in-production`, `dev-token`).

## Important conventions

- Go module: `github.com/zhinea/skylex`
- Requires **Go 1.26.1** per `go.mod`.
- UI is React Router 7 with **SSR enabled** (`ssr: true` in `react-router.config.ts`).
- Tailwind is v4 and loaded via `@tailwindcss/vite` in `ui/vite.config.ts`.
- `bin/`, `.vite/`, and `dist/` are gitignored; `make clean` removes `bin/` and `gen/`.
- `.kilo/plans/skylex-db-control-plane-plan.md` is a detailed design/plan document that predates much of the code; use it for architectural intent, but trust executable sources (Makefile, configs, source) for current behavior.

---
> Source: [zhinea/skylex](https://github.com/zhinea/skylex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
