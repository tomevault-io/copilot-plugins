## daz

> - Daz is a Go-based modular Cytube bot.

# AGENTS.md

## Repository Overview
- Daz is a Go-based modular Cytube bot.
- Source is organized under `cmd/`, `internal/`, and `pkg/`.
- Plugins live under `internal/plugins/`.
- Build output is `./bin/daz`.

## Build / Run
- Build: `make build` (runs `./scripts/build-daz.sh`).
- Run: `make run` (depends on `make build`).
- Direct run: `./bin/daz` after build.
- Config: copy `config.json.example` to `config.json`.

## Lint / Format
- Format: `make fmt` (runs `go fmt ./...`).
- Lint: `make lint` (runs `go vet ./...`).
- Manual gofmt: `gofmt -w <file.go>` when editing a file.

## Tests
- All tests: `make test` (runs `go test -race ./...`).
- Single package: `go test ./internal/plugins/greeter`.
- Single test name: `go test ./internal/plugins/greeter -run TestGetGreeting`.
- Single test with fresh cache: `go test ./internal/plugins/greeter -run TestGetGreeting -count=1`.
- Run a test across all packages: `go test ./... -run TestName`.
- Useful env: set `LOG_LEVEL=debug` to enable verbose logging.
- Live Cytube admin-plane/manual verification room: `daztest` (`https://cytu.be/r/daztest`).
- When a live test against a chat user is needed, use `daztester` in `daztest` rather than the production room.
- Default all future operator/admin-plane testing maneuvers to `daztest`; avoid using `always_always_sunny` for routine verification unless explicitly required.

## External Agent Rules
- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` found.
- For this repo, push with the GitHub identity `hildolfr` while dual-account work is active; do not accidentally push Daz changes as `segordon`.

## Project Decision Log
- 2026-06-30: For this project, chat is not currently treated as private/confidential. The requester explicitly stated that chat log leakage/visibility is acceptable unless requirements change. This applies to the `!bug` chat-reporting flow and its captured chat context (see `internal/plugins/commands/bug/plugin.go` and the deploy verification note in `docs/deploy-checklist.md`). No implementation change is requested solely to add chat-log privacy protections based on the current request. If future work handles privacy-sensitive content, revisit this assumption before treating chat or chat-derived logs/issues as confidential.

## Go Version
- Go version in `go.mod`: `go 1.25.9`.
- Toolchain: `go1.25.9` (if you use `GOTOOLCHAIN=auto`).

## Runtime Flags
- `-config` sets config file path (default `config.json`).
- `-health-host` controls the health/metrics bind host (default `127.0.0.1`).
- `-health-port` controls health server port.
- `-verbose` enables debug logging.
- Environment `FORCE_COLOR=1` forces colored logs.

## Imports
- Use standard Go import grouping (stdlib, third-party, local).
- Keep module-local imports prefixed with `github.com/hildolfr/daz/...`.
- Avoid import aliasing unless required to resolve conflicts.

## Formatting
- Always run `gofmt` on touched files.
- Keep lines reasonably short; wrap long literals or comments.
- Use tabs for indentation as produced by `gofmt`.

## Naming
- Exported identifiers use `CamelCase`.
- Unexported identifiers use `lowerCamelCase`.
- Package names are short, lowercase, no underscores.
- Use descriptive names; single-letter names only for trivial loops.
- Constants follow Go conventions; avoid screaming snake-case.

## Types and Structs
- Prefer explicit struct fields over anonymous structs.
- Use pointers for mutable/shared state.
- Keep config structs tagged with JSON tags.
- Use typed aliases only when they add clarity.

## Error Handling
- Return errors instead of panicking.
- Wrap errors with context: `fmt.Errorf("context: %w", err)`.
- In `main`, log errors and `os.Exit(1)` for fatal failures.
- For deferred cleanup, log close errors without overriding main errors.

## Logging
- Use `internal/logger` for app logging.
- Pass plugin/component name as the first argument.
- Use `logger.Debug` for verbose details and `logger.Info` for routine info.
- Avoid `log.Printf` except in CLI utilities that already use it.

## Concurrency
- Protect shared mutable state with `sync.Mutex` / `sync.RWMutex`.
- Use `context.Context` for cancellation of goroutines.
- Ensure goroutines exit on context cancellation or channel close.
- Prefer `sync.WaitGroup` for coordinated shutdown.

## Event Bus & Plugins
- Implement `framework.Plugin` interface for plugins.
- Register plugins via `framework.PluginManager`.
- Subscribe to event bus topics with `eventBus.Subscribe`.
- Use `eventbus` constants for core Cytube events.
- Keep plugin names stable; they are used in logs and routing.

### Command Plugin Contract (EventFilter)
- `command.register` must include a comma-separated `commands` list and stable `req.From` plugin name; EventFilter routes execution to `command.<plugin-name>.execute`.
- Always subscribe the plugin to `command.<plugin-name>.execute` before (or at least in the same startup phase as) command registration.
- Keep registration aliases and subscribed handlers in sync; if you register `sign`, `spin`, etc., verify they resolve to your plugin and execute successfully.
- Prefer one canonical command entry that matches plugin identity (for example `couchcoins` for plugin `couchcoins`) and keep legacy aliases additive.
- In handlers, rely on `commandData.Name` and params (`username`, `channel`, `rank`, `is_pm`, `is_admin`) rather than inferring behavior from event topic names.
- For PM-capable commands, honor `is_pm` in response routing; do not leak PM-triggered private responses into channel chat unless explicitly intended (for example public jackpot announcements).
- Registration/persistence side effects must not block command availability at startup; failures should log and degrade safely instead of disabling command dispatch.
- Add/maintain tests that cover command dispatch for the canonical command plus at least one alias, and include cooldown + PM routing behavior.

## Economy API Guide

The economy API is an EventBus request/reply contract for reading and mutating per-channel user balances. When present, the stable contract is documented in `docs/economy.md`. Callers should prefer the Go wrapper in `internal/framework/economy_client.go`.

### Implementation Locations
- Plugin handler and routing: `internal/plugins/economy/plugin.go`
- Store interface and SQL-backed store: `internal/plugins/economy/store.go`
- Caller wrapper client: `internal/framework/economy_client.go`
- SQL schema, functions, and ledger table: `scripts/sql/033_economy_api.sql`
- Migration application: `internal/plugins/sql/plugin.go` (`applyExternalMigrations`)
- Plugin registration: `cmd/daz/main.go`
- Example config stub: `config.json.example` (`"plugins": { "economy": {} }`)

### Request/Response Contract Basics
- Event type: `plugin.request`
- Request envelope: `framework.EventData.PluginRequest` (serialized as `plugin_request`)
- Response envelope: `framework.EventData.PluginResponse` (serialized as `plugin_response`)
- Required operation names (`PluginRequest.Type`):
  - `economy.get_balance`
  - `economy.credit`
  - `economy.debit`
  - `economy.transfer`

The economy plugin routes and validates using the payload only. Handlers do not receive `EventMetadata`.

### Correlation and Response Delivery (Required)
- `PluginRequest.ID` is the only handler-visible correlation identifier.
- If you use `eventBus.Request(...)`, set `EventMetadata.CorrelationID` to the same value as `PluginRequest.ID`, otherwise the caller will wait on an ID the handler cannot see.
- The economy plugin delivers responses via `eventBus.DeliverResponse(req.ID, resp, err)` and no-ops if `req.ID` is empty.

### Idempotency Rules (credit, debit, transfer)
- Mutations are idempotent per `(channel, idempotency_key)`.
- If `idempotency_key` is missing or empty, the server defaults it to `PluginRequest.ID`.
- Retrying the same idempotency key with an identical payload returns `already_applied=true`.
- Retrying the same idempotency key with a different payload fails with `error_code=IDEMPOTENCY_CONFLICT`.
- Clients should retry by re-sending identical JSON fields and values.

### Error Codes
- `INVALID_ARGUMENT`: missing required fields, invalid types, invalid JSON for `data.raw_json`, unknown operation name
- `INVALID_AMOUNT`: `amount` missing, not an integer, or less than or equal to 0
- `INSUFFICIENT_FUNDS`: debit or transfer would make the sender balance negative
- `IDEMPOTENCY_CONFLICT`: duplicate idempotency key with a different payload
- `DB_UNAVAILABLE`: DB layer not ready (for example SQL plugin down) or requests cannot be executed
- `DB_ERROR`: DB operation failed after the DB layer was available
- `INTERNAL`: unexpected error

### Quick Start (Go)

```go
client := framework.NewEconomyClient(bus, "myplugin")

bal, err := client.GetBalance(ctx, channel, username)
if err != nil {
    // handle err
}
_ = bal

_, err = client.Credit(ctx, framework.CreditRequest{
    Channel:  channel,
    Username: username,
    Amount:   250,
    Reason:   "daily_reward",
})

_, err = client.Debit(ctx, framework.DebitRequest{
    Channel:        channel,
    Username:       username,
    Amount:         50,
    IdempotencyKey: "purchase-123",
    Reason:         "shop_purchase",
})

_, err = client.Transfer(ctx, framework.TransferRequest{
    Channel:      channel,
    FromUsername: "alice",
    ToUsername:   "bob",
    Amount:       10,
    Reason:       "tip",
})
```

Notes:
- `EconomyClient` generates a correlation ID per call and mirrors it into `PluginRequest.ID`. For mutations it also defaults `idempotency_key` to that same value, unless you set one explicitly.
- Structured failures map to `*framework.EconomyError` (`ErrorCode`, `Message`, optional `Details`).

### SQL Surface Area
- Ledger table: `daz_economy_ledger` (credits, debits, transfers) with optional `idempotency_key` and `metadata`
- Functions:
  - `daz_economy_get_balance(channel, username)`
  - `daz_economy_credit(channel, username, amount, idempotency_key, actor, reason, metadata)`
  - `daz_economy_debit(channel, username, amount, idempotency_key, actor, reason, metadata)`
  - `daz_economy_transfer(channel, from_username, to_username, amount, idempotency_key, actor, reason, metadata)`

### Testing
- Unit tests: `go test ./internal/plugins/economy -count=1`
- Integration tests (build tag `integration`, requires PostgreSQL):
  - Env vars: `DAZ_DB_HOST`, `DAZ_DB_PORT`, `DAZ_DB_NAME`, `DAZ_DB_USER`, `DAZ_DB_PASSWORD`
  - Run: `go test -tags=integration ./internal/plugins/economy -run TestTransfer -count=1`

### Extending The API
1. Update contract docs: add the new operation to `docs/economy.md` with request/response payload shape and error semantics.
2. Add plugin routing: define a stable operation name constant in `internal/plugins/economy/plugin.go`, extend the `switch req.Type`, implement a handler that validates and normalizes inputs.
3. Add store and SQL function: extend `internal/plugins/economy/store.go` and implement a new SQL function in a migration under `scripts/sql/` (including explicit grants).
4. Wire the migration: ensure `internal/plugins/sql/plugin.go` applies the new migration on startup.
5. Add tests: unit tests for payload validation and error mapping, plus integration tests for idempotency and ledger effects.

## Configuration
- Load config via `config.LoadFromFile` (supports env overrides).
- Validate config with `cfg.Validate()` before use.
- Use `cfg.GetPluginConfig("<name>")` for plugin config.

## Database / SQL
- Prefer `framework.SQLClient` for plugin DB access.
- Close `rows` and `db` handles with deferred cleanup.
- Use `sql.Open` and `db.Ping` when connectivity must be verified.
- Wrap SQL errors with context and preserve original error.
- Any schema change (new table/column/index/constraint) MUST include an explicit migration path in the same PR (migration SQL and runtime wiring if needed); do not rely on ad-hoc startup DDL alone.

## Tests Style
- Use the standard `testing` package.
- Use table-driven tests with `t.Run` for multiple cases.
- Use `t.Fatalf` for setup failures; `t.Errorf` for per-case failures.
- Clean up temp files with `defer` and ignore cleanup errors.
- Avoid network or real database usage unless test explicitly requires it.

## File Organization
- `cmd/daz/` contains CLI entrypoints.
- `internal/` contains core framework and plugins.
- `pkg/` contains shared packages for external use.
- Tests live alongside implementation files as `*_test.go`.

## Docs
- Use existing docs under `docs/` when updating behavior.
- Update `README.md` if commands or configuration change.
- Keep examples in `config.json.example` in sync with code.

## Documentation Canon
- `AGENTS.md` is a canonical operational/source-of-truth document for LLM-directed workflows and must be retained.
- Canonical project docs live in top-level `docs/*.md` (for example architecture, API, plugins, setup, DB schema, economy, and performance notes).
- Historical plans, temporary notes, and one-off validation documents must go under `docs/archive/`.
- When behavior/API/schema changes, update canonical docs in the same revision.
- Do not remove existing AGENTS guidance; only additive clarifications are allowed.
- When GitHub CI is unavailable, run lightweight local doc hygiene checks before push (for example validate internal links and ensure moved/renamed docs are reflected in `README.md` and `docs/README.md`).

## Common Tasks
- Add a plugin: implement `framework.Plugin` and register in `cmd/daz/main.go`.
- Add a command plugin: follow `internal/plugins/commands/*`.
- Add an event handler: register with `eventBus.Subscribe`.

## Safety Checks
- Avoid committing secrets; `config.json` should not be checked in.
- Use `config.json.example` for sample values.
- Prefer environment variables for local credentials.

## Pull Request Hygiene
- Keep changes focused and minimal.
- Run `make fmt` and `make test` when feasible.
- Mention any DB or external dependencies in notes.
- Create a detailed commit and push after every single revision or code change.

## Git Credentials
- Deploy key: `./dazza_deploy_key` (private) and `./dazza_deploy_key.pub` (public).
- Git username: `hildolfr`.
- Git email: `svhildolfr@gmail.com`.
- Push note: set `origin` as the write target (`https://github.com/hildolfr/daz.git`) and push feature branches there (for example `git push -u origin feat/...`), since other remotes may be read-only/permission-restricted.

### HTML Pages + Plugin Extensions (help/gallery)
- Both help and gallery generators push to `origin` on `gh-pages` and use `GITHUB_TOKEN` or `dazza_deploy_key`.
- Each generator protects its working tree with a marker:
  - help: `data/help/.daz_help_marker`
  - gallery: `data/galleries/.daz-gallery`
- To add a new plugin HTML surface without clobbering others:
  1. Generate output only under its own directory (for example `data/help/<plugin>/` or `data/galleries/<plugin>/`).
  2. Ensure the generator writes its marker file and only `git add` that plugin subtree (not `git add -A` at repo root).
  3. Keep pushes scoped to `gh-pages` and let the existing `gh-pages` checkout/rebase flow handle merge with current remote state.
  4. If a marker is missing in an existing git directory, fail fast and do not auto-delete/overwrite unrelated files.

## When Unsure
- Read similar plugins under `internal/plugins/` first.
- Follow existing patterns for error wrapping and logging.
- Ask for clarification if behavior affects multiple plugins.

## Quick Reference Commands
- `make build`
- `make run`
- `make fmt`
- `make lint`
- `make toolchain-check`
- `make static-check`
- `make static-check-changed`
- `make test`
- `make docs-check`
- `make migration-check`
- `make smoke-command-path`
- `make verify-runtime`
- `make predeploy-check`
- `make release-smoke`
- `go test ./internal/... -run TestName`

## Local Runtime (prefer user systemd unit)
- Primary deployment target is the remote host, not this development machine. Treat `daz.service` on the remote host as the default production service, and use `deploy/systemd/daz.service` as the versioned starter unit.
- The update plugin/config example default to `systemctl restart daz.service`; override `plugins.update.restart_command` on the deployed host if you need `sudo`, a wrapper, or a different service name.
- Use the user unit for running/restarting locally; avoid ad-hoc shell launch commands for daz.
- Service file: `~/.config/systemd/user/dazza-dev.service`
- Rebuild and restart flow:
  - `make build`
  - `systemctl --user daemon-reload`
  - `systemctl --user restart dazza-dev.service`
- Useful status/debug:
  - `systemctl --user status dazza-dev.service --no-pager`
  - `journalctl --user -u dazza-dev.service -f`
- Optional stop/start:
  - `systemctl --user stop dazza-dev.service`
  - `systemctl --user start dazza-dev.service`
- Do not use the system-level unit for local dev/test unless explicitly running as a system service.

## Legacy User-State Schema (032)
- Migration file: `scripts/sql/032_user_state_foundation.sql`.
- Runtime application: `internal/plugins/sql/plugin.go` via `applyExternalMigrations()` in `initializeSchema()`.
- Purpose: storage foundation for legacy Dazza-style user quantities before plugin porting.
- Economy table: `daz_user_economy` (`balance`, `trust_score`, `heist_count`, `last_heist_at`, `total_earned`, `total_lost`).
- Body-state table: `daz_user_bladder` (`current_amount`, `last_drink_time`, `drinks_since_piss`, `last_piss_time`).
- Pissing contest tables: `daz_pissing_contest_stats`, `daz_pissing_contest_challenges`.
- Game stat tables: `daz_mystery_box_stats`, `daz_couch_stats`, `daz_coin_flip_stats`, `daz_sign_spinning_stats`, `daz_mug_stats`.
- Progression/session tables: `daz_coin_flip_challenges`, `daz_bong_sessions`, `daz_user_bong_streaks`.
- All new tables are channel-scoped (`channel` + `username` uniqueness), include timestamps, and grant CRUD access to `daz_user`.

## Footnotes
- Build script is `scripts/build-daz.sh` and is authoritative.
- The build script creates `./bin/daz` and enforces location checks.
- The bot expects Postgres 14+ per README.
- The repo uses Go modules; run `go mod download` if needed.
- Keep log levels configurable via config/env and `logger.SetDebug`.

## Player State API (bladder/alcohol/weed/food/lust)
- Use `framework.NewPlayerStateClient` in command/game plugins.
- API client methods for whole-state:
  - `Get(ctx, channel, username)` → `PlayerState`
  - `Set(ctx, PlayerStateMutationRequest)` (sparse overwrite)
  - `Adjust(ctx, PlayerStateMutationRequest)` (deltas)
- Per-field helper methods:
  - `SetBladder`, `AdjustBladder`
  - `SetAlcohol`, `AdjustAlcohol`
  - `SetWeed`, `AdjustWeed`
  - `SetFood`, `AdjustFood`
  - `SetLust`, `AdjustLust`
- All values are clamped at the store layer to never go below zero.
- Example usage:
  - `stateClient := framework.NewPlayerStateClient(bus, "myplugin")`
  - `stateClient.AdjustBladder(ctx, channel, username, -5)` for bladder drain
  - `stateClient.AdjustLust(ctx, channel, username, 12)` for lust gain

---
> Source: [hildolfr/daz](https://github.com/hildolfr/daz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
