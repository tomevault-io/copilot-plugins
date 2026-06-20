## galaxy-os

> This file provides guidance to AI agents when working with code in this repository.

# CLAUDE.md

This file provides guidance to AI agents when working with code in this repository.

---

## Project

Reusable Discord bot platform designed to deploy multiple bots across multiple servers. The architecture provides a shared base layer for common concerns (config, services, features) so that each bot is a thin composition of features on top of the platform.

Currently deployed on the Galaxy One server with two bots: `hue` and `kevin`.

---

## Tech Stack

Go project — see `go.mod` for current versions of all dependencies.

| Library                 | Role                                                                 |
| ----------------------- | -------------------------------------------------------------------- |
| `disgoorg/disgo`        | Discord framework (gateway, REST, sharding, OAuth2, command handler) |
| `spf13/viper`           | Config management with Sub-tree access for nested feature configs    |
| `spf13/cobra`           | CLI structure (`bot start`, `generate`)                              |
| `robfig/cron/v3`        | Cron scheduling with timezone support                                |
| `disgoorg/snowflake/v2` | Discord snowflake ID type                                            |
| `jackc/pgx/v5`          | PostgreSQL connection pool                                           |
| `aws/aws-sdk-go-v2`     | Email via SES                                                        |

---

## Build and Run

```sh
make run/<name>       # Build and run bot <name>
make build/<name>     # Build only
make test             # go test -v -race -buildvcs ./...
make audit            # Tests + vet + govulncheck + deadcode
make tidy             # go mod tidy + go fmt
```

The `bot` name is injected at compile time via `-ldflags` into `cmd.bot`. It determines which config section to read (`[bot.<name>]`) and which feature set to activate (defined in `cmd/bots.go`'s `bots` map). It is not a runtime argument.

---

## Architecture

### Package structure

```
conf/              # TOML config + secrets (secrets gitignored, config committed)
cmd/               # CLI entry + boot composition
  bot.go           # "bot start" command + stage pipeline
  bots.go          # botDef + bots map
  registrar.go     # muxRegistrar adapter
feature/           # Top-level feature catalog — one sub-package per feature
  <name>/
    feature.go     # entrypoint: var Feature + Setup/Start/Stop
    *.go           # helper types/logic
internal/
  core/            # Framework core — interfaces, boot pipeline, lifecycle
  config/          # Config implementation (FileStore, Resolver, Provider)
  discord/         # Discord adapters (RestAdapter)
  guild/           # Guild lifecycle manager
  i18n/            # Internationalization (Text, LocaleResolver)
  service/         # Shared service implementations (cron, oauth, email, sql, logger)
.gen/              # All generated/build artifacts (gitignored)
  bin/             # Compiled binaries
  reports/         # Audit, coverage reports
```

### Dependency rules

- `core/` imports only stdlib + disgo types. Never imports feature/, service/, config/
- `feature/` imports only `core/` (interfaces). Never imports config/, service/, or other features
- `service/` imports only `core/` (interfaces). Never imports feature/ or config/
- `config/` imports only `core/` (interfaces)
- `guild/` imports only `core/` (interfaces)
- `cmd/` is the sole composition root — imports everything

### Boot pipeline

All initialization runs through a declarative stage pipeline in `cmd/bot.go` via `core.Run()`:

1. **config** — loads `config.toml` + merges `secrets.toml`, parses Global/Bot/Log
2. **logger** — initializes structured slog
3. **discord** — creates disgo bot.Client, sets up handler router
4. **guilds** — queries Discord API, loads per-guild config, initializes GuildManager
5. **services** — demand-driven activation: scans all feature `Needs()`, starts only required services
6. **features** — builds per-feature `Deps`, calls `Setup(deps)` then `Start(ctx)` per scope
7. **gateway** — opens Discord gateway connection
8. **ready** — waits for SIGINT/SIGTERM, triggers graceful shutdown

Shutdown is reverse order: features Stop, services StopAll, gateway Close.

### Feature system

Each feature is a struct implementing `core.Feature`:

```go
type Feature interface {
    Name() string
    Scope() Scope                    // BotScope | GuildScope | CrossGuildScope
    Needs() []ServiceID              // demand-driven service activation
    Setup(deps Deps) error           // wire dependencies
    Start(ctx context.Context) error // begin active work
    Stop(ctx context.Context) error  // graceful teardown
}
```

Features receive dependencies via `core.Deps` — a struct built per-feature by the framework:

- Always populated: Logger (scoped), Rest (RestClient), Commands (Registrar), Locale, Configs (ConfigProvider), Bus (InnerBus), Env, BotName, Gateway (DiscordGateway — event listeners, presence, bot identity)
- Scope-dependent: GuildID (GuildScope), Guilds (CrossGuildScope)
- Needs-dependent: Cron (only if CronService declared), OAuth (only if OAuthService declared)

**Adding a new feature:**

1. Create a sub-package in `feature/<name>/`
2. Define the entrypoint in `feature.go`: a struct implementing `core.Feature` with a config struct implementing `core.Config`
3. Export a package-level var: `var Feature = &myFeature{}`
4. Register in `cmd/bots.go`'s `bots` map for the relevant bot(s)

### Configuration system

TOML-based, managed by viper. Config and secrets are separated:

- `conf/config.toml` — global + per-bot non-sensitive config (`[bot.<botname>]`, `[features.<botname>.<featurekey>]`). Committed.
- `conf/config.<guild_id>.toml` — per-guild overrides with inheritance from bot defaults. Committed.
- `conf/secrets.toml` — tokens, credentials, API keys (`[bot.<botname>]` secret fields). Gitignored.
- `conf/secrets.<guild_id>.toml` — per-guild secrets. Gitignored.

Secrets are merged on top of config at load time via viper. Config access in features via `core.ConfigProvider`:

```go
core.ResolveGuild[MyConfig](deps.Configs, guildID)  // guild-scoped
core.ResolveBot[MyConfig](deps.Configs)              // bot-scoped
```

### Services layer

Each service implements `core.Service` (Name/Start/Health/Stop). Services are demand-activated — they start only when a feature declares them in `Needs()`. Current services: CronService, OAuthService, SQLService (deferred), EmailService (deferred).

### Localization

`internal/i18n` provides `i18n.Text` (`map[discord.Locale]string`) for Discord-facing localized strings. Always provide at least `discord.LocaleEnglishUS`.

---
> Source: [bil0u/galaxy-os](https://github.com/bil0u/galaxy-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
