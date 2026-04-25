## telegram-ogn-tracker

> Telegram bot for real-time paraglider/glider tracking via OGN (Open Glider Network) APRS.

# telegram-ogn-tracker

Telegram bot for real-time paraglider/glider tracking via OGN (Open Glider Network) APRS.

## Structure
```
cmd/bot/main.go              — entry point
internal/tracker/
  tracker.go                 — Tracker struct, init, types (TrackInfo, Coordinates)
  client.go                  — OGN client connection, position updates goroutine
  commands.go                — Telegram command handlers
```

## Dependencies
- `ogn` (alias for `github.com/evtaccount/ogn-client`) — APRS client & parser
- `github.com/go-telegram/bot v1.20.0` — Telegram API

## Architecture
- **Tracker** — main struct: bot, APRS client, tracking map (`map[string]*TrackInfo`), mutex, session state, landing coords
- **TrackInfo** — per-glider state: MessageID, TextID, Position (`*parser.PositionMessage`), Name, Username, LastUpdate
- `/track_on` spawns 2 goroutines:
  1. `runClient(stopCh)` — receives OGN beacons via APRS, parses, updates tracking map
  2. `sendUpdates(stopCh)` — every 30s sends/edits Telegram live location messages
- Concurrency: `sync.Mutex` on shared state, `stopCh` channel for graceful shutdown

## Commands
| Command | Effect |
|---------|--------|
| `/start` | Reset, welcome |
| `/start_session` | Enable session, clear tracking |
| `/add <id> [name]` | Add glider (matches last 6 chars of callsign) |
| `/remove <id>` | Remove glider |
| `/track_on` | Start OGN + update goroutines |
| `/track_off` | Stop goroutines, keep tracking list |
| `/session_reset` | Stop all, clear everything |
| `/landing` | Set landing location (2min timeout, Haversine distance) |
| `/list` | Show tracked IDs |
| `/status` | Tracking on/off + count |
| `/help` | List commands |

## APRS Filter Logic
- IDs ≥7 chars → server-side `b/ID1/ID2/...` filter
- Any ID <7 chars → no filter, client-side matching by last 6 chars

## Build & Run
- `make run` — local dev (`go run ./cmd/bot`)
- `make build` / `make up` — Docker
- `make rebuild` — full Docker rebuild
- CI: GitHub Actions on `v*` tags → multi-platform binaries + GHCR image

## Config
- `TELEGRAM_BOT_TOKEN` env var (from `.env`, see `.env.example`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evtaccount) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
