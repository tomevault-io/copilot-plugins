## quant-ba

> > 面向用户的使用说明见 [docs/USAGE.md](file:///Users/colinmyth/code/quant_ba/docs/USAGE.md)。

# AGENTS.md

> 面向用户的使用说明见 [docs/USAGE.md](file:///Users/colinmyth/code/quant_ba/docs/USAGE.md)。
> 本文件保留面向开发者的项目结构与约定。

## Project
`quant_ba` is a Go quantitative trading system for Binance with backtest, paper, and live modes. Strategies run as separate plugin binaries communicating with the host over JSON-RPC on stdin/stdout.

## Layout
- `cmd/quant_ba/` — CLI entrypoint (`main.go` -> `Execute()` -> `rootCmd` from `root.go`). Each subcommand lives in its own file (`backtest.go`, `paper.go`, `live.go`, `serve.go`, `strategy.go`, `configCmd` in `root.go`).
- `internal/` — host packages: `config`, `store` (SQLite), `market` (REST + WS), `order` (paper + live HMAC), `portfolio`, `risk`, `backtest`, `executor`, `api`, `strategy` (host-side loader/protocol), `types`, `indicator`.
- `internal/strategy/<name>/main.go` — strategy plugin binaries. Built into `plugins/<id>`.
- `internal/strategy/pluginkit/` — boilerplate (`pluginkit.Run`, `Closes`, `HasPosition`) used by plugins.
- `internal/strategy/example/ma_cross.go` — older hand-rolled example (does NOT use pluginkit). Prefer the pluginkit pattern for new plugins.
- `config/default.yaml` — runtime config (exchange URLs, risk limits, store path, server port). API keys default empty.
- `data/` and `results/` — gitignored runtime artifacts.
- `docs/superpowers/{plans,specs}/` — design history (not normative).

## Build / run
- Go 1.20, module `github.com/colinmyth/quant_ba`. No Makefile; use `go` directly.
- Build host binary: `go build -o quant_ba ./cmd/quant_ba` (or `./quant_ba` already exists at repo root).
- Build a plugin: `go build -o plugins/<id> ./internal/strategy/<name>` — e.g. `go build -o plugins/bollinger_v1 ./internal/strategy/bollinger`.
- Run: `./quant_ba --help`, then e.g. `./quant_ba backtest run plugins/bollinger_v1`, `./quant_ba paper start plugins/bollinger_v1`, `./quant_ba serve`, `./quant_ba live start plugins/<id>`.
- Config flag: `-c /path/to.yaml` (default `config/default.yaml`). `app.go:42` also `os.MkdirAll("data", 0755)` relative to CWD, so run from repo root or pass `-c` with a relative-safe path.

## Tests / verify
- `go test ./...` — runs existing suites (`backtest`, `risk`, `portfolio`, `indicator`, `executor`).
- No separate lint config; `go vet ./...` is the convention used in this repo.
- The `.claude/settings.local.json` shows the working command set; mirror those when verifying.

## Plugin protocol (host <-> plugin)
- JSON-RPC, one object per line on stdin/stdout. Messages defined in `internal/strategy/protocol.go`.
- Methods: `meta`, `init`, `bar`, `order_update`, `pause`, `resume`, `stop`. Plugin returns `{}` for the lifecycle ones; `meta` returns `MetaResult`; `bar` returns `SignalResult{Signal *types.Signal}` (nil/hold = no action).
- New plugin skeleton (`internal/strategy/bollinger/main.go` is the canonical template):
  - Implement `pluginkit.Strategy` (`Meta()` + `OnBar(OnBarParams) *types.Signal`). Optional `pluginkit.Initializer` for `OnInit`.
  - `main()` is `pluginkit.Run(&strat{})`.
  - Use `pluginkit.Closes(bars)` for close prices and `pluginkit.HasPosition(sym, positions)` for position checks.
- Signals: `Size: 0` means "close whole position" — the executor resolves it from the current holding. Market orders should leave `Price` 0; `executor/paper.go:referencePrice` fills it from the latest bar close for risk/portfolio accounting.
- Loader (`internal/strategy/loader.go`) starts the binary, calls `meta` to register, then exposes `Client` for `bar`/`init`/`stop`. The host kills the process on `Loader.Close`.

## Conventions
- One-way net position model (Binance one-way mode). `DirBuy` = long exposure, `DirSell` = short exposure; `Size: 0` closes.
- Spot vs futures is selected by `portfolio.New(store, futures)`. In futures mode, `Equity()` and `OnOrderFilled` use margin-based accounting with `Order.Leverage`. The current uncommitted work adds this; `cmd/quant_ba/app.go:61` was not updated and still calls `portfolio.New(store)` — needs the `futures` arg to compile.
- Fill modeling: backtest (`backtest/fill.go`) and paper (`order/paper.go`) share slippage/fee constants; paper applies `paperSlippageBuy/Sell` and `paperFeeRate`.
- Backtest bars are revealed bar-by-bar to the plugin via a `seen` slice to prevent lookahead bias (`internal/backtest/engine.go:92`).
- SQLite is the only store. `db.SetMaxOpenConns(1)`. Schema migrates on open in `store/sqlite.go:migrate`.
- Logging: `log.Printf` to stderr in `executor/paper.go`. CLI commands print to stdout.

## Gotchas
- `quant_ba`, `data/`, `results/`, `plugins/`, `.DS_Store` are gitignored. The committed `plugins/*` binaries in the current tree are stale from prior runs; rebuild before testing.
- The WebSocket dialer uses `http.ProxyFromEnvironment`; on networks where Binance is blocked, `export https_proxy=http://127.0.0.1:7897 http_proxy=http://127.0.0.1:7897` is the established workaround (see `.claude/settings.local.json`).
- `internal/strategy/example/ma_cross.go` is a legacy hand-rolled RPC handler, not a `pluginkit` example. Don't copy it for new plugins.
- Live trading requires real `api_key`/`secret` in `config/default.yaml`; defaults are empty. Use `./quant_ba paper start` to exercise the loop without keys.
- Server endpoints: `GET /health`, `GET /api/strategies`, `GET /api/portfolio`. Port from `server.port` (default 8080), force-enabled in `serve` subcommand.
- `cmd/quant_ba/live.go:liveStartCmd` is a stub — it loads the plugin and prints a notice, but does not run the executor loop. Only `paper start` and `backtest run` are end-to-end today.

---
> Source: [mythc/quant_ba](https://github.com/mythc/quant_ba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
