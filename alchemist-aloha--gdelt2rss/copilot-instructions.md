## gdelt2rss

> Single-package Go (`package main`, module `gdelt2rss`, Go 1.27). No lint/CI/Makefile.

# AGENTS.md

Single-package Go (`package main`, module `gdelt2rss`, Go 1.27). No lint/CI/Makefile.

## Commands
- `cp config.example.yaml config.yaml` — `config.yaml` is gitignored, never edit in place for examples.
- `go run . -config config.yaml` — default flag is `/config/config.yaml` (container path), must override locally.
- `go test ./...` / `go vet ./...` — currently `TestFetchRSSStatusAndRetention` FAILS: fixture date `20260830T120000Z` is older than `cache.max_age: 48h` relative to now, so the article is pruned before the RSS assert.
- Compose: `docker compose up` (read-only, `9494:8080`); `-f compose.zero-write.yaml` for zero writes; `-f compose.retention.yaml` only with `retention.enabled: true` + `backend: sqlite` + `path: /data/gdelt-rss.db`.

## Config gotchas (`config.go`)
- `decoder.KnownFields(true)` — unknown keys error (see `TestConfigRejectsUnknownItems`).
- `Duration` supports extra `Xd` suffix and preserves raw text (`String()` returns original) because GDELT `timespan` is sent verbatim via `requestURL`.
- Per-source `search.raw_query` and structured `keywords`/`operators` are mutually exclusive; empty `request.*` fields inherit from `gdelt.defaults` via `mergeRequest`.
- `retention.enabled: false` → `newRetention` returns `(nil, nil)`; all callers nil-check (`main.go`, `server.go`, `retention.Enqueue/Run`). Enabling requires `backend: sqlite` (pure-Go `modernc.org/sqlite`, single conn, WAL).

## Runtime wiring
- `main.go:runPollers` — one goroutine per source with per-source `poll_interval` ticker, plus a global mutex enforcing `gdelt.scheduler.min_request_interval` across all sources; `max_retries`/`retry_backoff` inside poll.
- `server.go:Handler` — `GET /feeds/{id}.xml` (`.xml` suffix required, cached `rss.cache_duration`), `GET /api/v1/status` (`ok`→`degraded` if any source `LastError` or retention error), `GET /` index.
- `cache.go` — RAM-only; `canonicalURL` strips fragment, `utm_*`/`fbclid`, lowercases host; prune order is `max_age` then oldest-first to `max_articles` (`max_age: 0` = unbounded but hard cap still applies).

---
> Source: [Alchemist-Aloha/gdelt2rss](https://github.com/Alchemist-Aloha/gdelt2rss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
