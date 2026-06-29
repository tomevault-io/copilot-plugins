## kafka-king

> Kafka-King: a cross-platform Kafka GUI client built with **Wails v2** (Go backend + Vue 3 frontend + Naive UI).

# AGENTS.md

## Project overview
Kafka-King: a cross-platform Kafka GUI client built with **Wails v2** (Go backend + Vue 3 frontend + Naive UI).
Uses [franz-go](https://github.com/twmb/franz-go) as the Kafka client library.

## Development commands

All commands below assume working directory is `app/` (the Wails project root, **not** the repo root).

```bash
cd app

# Development (hot-reload frontend + backend)
wails dev

# Build production binary
wails build

# Build with version injection (as CI does)
wails build -ldflags="-X 'app/backend/common.Version=v1.0.0'"

# Frontend dev only (Vite at http://localhost:5174)
npm --prefix frontend run dev

# Go tests (require a running Kafka cluster)
go test ./...
```

## Architecture

### Directory layout
```
repo-root/
  app/                    # <-- Go module "app" (Wails project root)
    main.go               # Entrypoint, registers Wails bindings
    wails.json            # Wails config (app name, dev server URL)
    frontend/             # Vue 3 + Vite + Naive UI
      src/
        App.vue           # Root layout, theme/lang switching
        components/       # Feature components (Conn, Topics, Consumer, etc.)
        utils/eventBus.js # mitt-based global event bus
        i18n/             # vue-i18n translations (zh-CN, en-US, ja-JP, ko-KR, ru-RU)
    backend/
      app.go              # App lifecycle (Start, DomReady, Shutdown)
      common/vars.go      # Constants, GitHub URLs, Version (injected at build)
      config/app.go       # Config read/write (~/.kafka-king/config.yaml), file dialogs
      service/kafka.go    # All Kafka operations (connect, topics, consume, produce, ACLs)
      system/update.go    # GitHub release update checker
      types/resp.go       # Request/response structs, Connect, Config, AlertConfig
      utils/              # compress (gzip/lz4/snappy/zstd), Ternary utility
```

### How the app is wired
1. `main.go` creates `App`, `AppConfig`, `Update`, `KafkaService` structs and binds them to Wails.
2. The frontend calls Go methods via auto-generated bindings in `frontend/wailsjs/go/`.
3. All bound Go methods return `*types.ResultResp` or `*types.ResultsResp` with an `Err` string field.
4. Backend methods are exposed by being struct methods — Wails auto-generates JS bindings from them.

## Important conventions

- **Go module root is `app/`**, not repo root. All `go` commands must run from `app/`.
- The `common.Version` variable is set at **build time** via ldflags. At dev time it is empty string.
- Config is persisted as YAML at `~/.kafka-king/config.yaml`. The `Config.Connects` field stores saved cluster connections (including passwords in plain text — never commit config.yaml).
- The `consumer` cache on `Service` stores `[group, topic, isolationLevel, *kgo.Client]` — the client pointer is at **index 3**, not index 2.
- The frontend uses a `mitt` event bus (`emitter`) for cross-component communication (theme change, menu select, language change).

## Testing

- Only `app/backend/service/kafka_test.go` exists, with hardcoded broker addresses.
- Tests are **integration tests** — they require a running Kafka cluster. No mocks.
- Run a single test: `go test -run TestConsume ./backend/service/`

## Build/CI

- CI triggers on **`release: created`** only (not on push or PR).
- CI injects the release tag as version: `-ldflags="-X 'app/backend/common.Version=${{ github.ref_name }}'"`
- Linux build requires system dependencies: `libgtk-3-dev libwebkit2gtk-4.0-dev build-essential`
- Node version: 22.x, Go version: 1.24

## Key libraries

| Library | Purpose |
|---------|---------|
| `github.com/twmb/franz-go` | Kafka client |
| `github.com/wailsapp/wails/v2` | Desktop app framework |
| `vue-i18n` | Frontend i18n |
| `naive-ui` | Vue 3 component library |
| `@vicons/material` | Icon set |
| `mitt` | Lightweight event bus |
| `echarts` | Charts (monitor page) |
| `go-resty/resty/v2` | HTTP client (update check, ping, webhook alerts) |
| `github.com/jcmturner/gokrb5/v8` | Kerberos/GSSAPI SASL |
| `github.com/klauspost/compress` | Snappy and zstd compression |
| `github.com/pierrec/lz4/v4` | LZ4 compression |

## Gotchas

- `.gitignore` excludes `frontend/dist/` and `frontend/wailsjs/` — these are generated at build time. Run `wails build` or `wails dev` once to populate them.
- The `frameless` window uses `--wails-draggable:drag` CSS on the sidebar for custom title bar dragging.
- Connection struct fields map to specific Kafka config options; `tls=="enable"` and `sasl=="enable"` are string comparisons, not booleans (this is because Wails serializes JS-to-Go as `map[string]any`).
- The app pings `https://ysboke.cn/api/kingTool/ping` on startup for anonymous version tracking.

---
> Source: [Bronya0/Kafka-King](https://github.com/Bronya0/Kafka-King) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
