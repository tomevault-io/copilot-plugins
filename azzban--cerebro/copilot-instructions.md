## config-conventions

> Config loading, validation, and usage conventions for Cerebro


# Config Conventions

## Four-File Model

| File | Contains | Tracked in Git? |
|---|---|---|
| `configs/app.yaml` | Engine, risk, TUI, ingest, agent settings | ✅ (sanitised) |
| `configs/markets.yaml` | Venues, symbols, leverage | ✅ |
| `configs/strategies.yaml` | Strategy parameters | ✅ |
| `configs/secrets.env` | API keys, DB URL, bot tokens | ❌ (gitignored) |

Always edit `*.example` files alongside the real files when adding new config fields.

## Struct Conventions

- All config structs live in `internal/config/`.
- Nested structs mirror YAML hierarchy (e.g. `Config.Engine.KillSwitch`).
- Secret values loaded from env via `godotenv`; **never hardcode them in YAML**.
- Every new field needs a validation check in `config.Validate()`.

## Accessing Config

Config is passed **once** through `internal/app.New(cfg)` and stored on the `App` struct. Downstream packages receive **only the fields they need** — do not pass the full `*config.Config` deep into domain logic.

```go
// ❌ BAD — passes whole config into a strategy
strategy.New(cfg)

// ✅ GOOD — pass specific values
strategy.New(cfg.Strategy.RSIPeriod, cfg.Strategy.BBWidth)
```

## Environment Safety

`Environment` must be one of `paper` | `production`. The CLI enforces triple-agreement at startup:

1. `ENVIRONMENT=production` in `secrets.env`
2. `environment: production` in `app.yaml`
3. `--live` flag passed to `run`

Any mismatch is a **fatal validation error** — do not weaken these checks.

## Makefile / Docker

`DATABASE_URL` and other secrets in `Makefile` must only ever be set via environment variables or a local `.env` override, never hardcoded in the file that gets committed.

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
