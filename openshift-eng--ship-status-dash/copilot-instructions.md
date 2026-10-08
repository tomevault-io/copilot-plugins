## dev-commands

> Common dev commands for database migration, linting, and testing


### Database migration

Run migrations: `go run ./cmd/migrate --dsn "$SHIP_STATUS_DSN"`

If `SHIP_STATUS_DSN` is not set, use the dev default: `postgres://postgres:password@localhost:5433/ship_status?sslmode=disable`

### SLO seed

After migrate, local dev loads sample payload workspace rows:

`go run ./cmd/seed-slo --dsn "$SHIP_STATUS_DSN" --config hack/local/dashboard/config.yaml`

`--dsn` is required. `--config` defaults to `hack/local/dashboard/config.yaml`. `hack/local/dashboard/local-dev.sh` runs this after migrate. E2e passes `test/e2e/scripts/dashboard-config.yaml`.

### Linting

Run lint: `make lint`

The lint script uses `golangci-lint` directly when available, falling back to a container otherwise.

### Testing

Run unit tests: `make test`

Run frontend BDD tests: `make bdd`

Run e2e tests: `make local-e2e`

### Frontend

Install dependencies: `cd frontend && npm ci --no-audit --ignore-scripts`

Start dev server: `cd frontend && npm run start`

Run lint/format: `cd frontend && npx eslint . --fix && npx prettier --write .`

### APM

Regenerate agent context (rules, commands, `AGENTS.md`): `make apm`

Requires **uv** / **uvx** (preinstalled in the devcontainer). `make verify-apm` regenerates and fails if those outputs differ from HEAD. That is the CI check. `make lint` does not run it.

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
