## cerebro

> Quick commands and tribal-knowledge notes that don't fit inside per-package docs.

# Cerebro — Agent / Operator Notes

Quick commands and tribal-knowledge notes that don't fit inside per-package docs.
See `CLAUDE.md` for the canonical architecture + conventions.

## Verification

```bash
make build         # build binary
make test          # unit tests
make lint          # golangci-lint
make check         # dry-run config validation
```

## CryptoPanic news scraper

Scraper lives at `internal/ingest/news/cryptopanic/`. Two tiers:

1. **RE (primary)** — `client.go` + `crypto.go`. Pure-Go AES-128-CBC + zlib
   decrypt of the `/web-api/posts/` endpoint. ~200ms per scrape.
2. **Browser (fallback)** — `browser.go`. Headless Chromium that hooks
   `JSON.parse` and captures the decrypted payload. ~5s per scrape.

`fallback.go` composes the two with a circuit breaker: after 3 consecutive
`ErrBadPayload` failures from the RE path (key rotation, IV formula
change, etc.), it cools down for 1 hour and routes all traffic through
the browser. A single `system_alerts` notification is sent per cool-down.

### When the alert fires: refresh the AES key

```bash
scripts/extract_cryptopanic_key.sh
```

The script:
1. Fetches `https://cryptopanic.com/` and locates the current
   `cryptopanic.min.*.js` bundle.
2. Extracts the packed `dk()` function and executes it in Node to resolve
   the 16-byte AES key.
3. Prints the new key + bundle hash so you can diff against `key.go`.

If the key changed:
1. Update `const aesKey` and `const sourceBundleHash` in
   `internal/ingest/news/cryptopanic/key.go`.
2. Capture a fresh ciphertext/plaintext pair in
   `internal/ingest/news/cryptopanic/testdata/` (see the node script at
   `/tmp/verify.js` during development, or write a small capture helper).
3. `go test ./internal/ingest/news/cryptopanic` must pass.

### Live smoke test

The `live` build tag lets you hit the real CryptoPanic site:

```bash
go test -tags=live ./internal/ingest/news/cryptopanic -run TestLive -v
```

Expect ~50 posts in ~1.5s when the RE path is healthy.

### Cache layout

The ingest scheduler writes to Redis on each tick:
- `news:latest` — global newest-first list, TTL = interval × 3
- `news:by_asset:<CODE>` — per-currency list for each code listed in
  `ingest.cryptopanic.currencies`, same TTL

The agent `fetch_latest_news` tool reads these keys first and only falls
back to a live scrape on cache miss.

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
