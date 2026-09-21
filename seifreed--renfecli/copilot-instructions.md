## renfecli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commits

Do **not** add `Co-Authored-By: Claude …` (or any AI co-author trailer) to
commits in this repository, and do not add a "Generated with" line to pull
request descriptions. The commit message should describe the change; who or
what typed it is not part of the record here.

## Commands

```bash
make build                     # ./renfe
make check                     # gofmt + go vet + go test + build + golangci-lint
go test ./internal/client -run TestDecodeDWR -v   # single test
golangci-lint run ./cmd/... ./internal/...   # CI runs this as a separate job
```

`make check` runs the linter too (skipping it with a warning when it is not
installed), so it matches what CI enforces. Run `govulncheck ./...` as well when
dependencies change.

## What this is

An unofficial Go client + CLI for `venta.renfe.com` (Spanish rail booking). It
replays the web app's own HTTP shapes; there is no documented API and no JS
engine in the loop. Read-only: it searches, prices and reads the signed-in
account, and never books or pays.

## Architecture

Dependencies point one way: `cmd/renfe` → `internal/client` (the library, which
depends on nothing else in the repo) and `internal/config` → `internal/store`.

`internal/client` is the library; `cmd/renfe` is a stdlib-`flag` CLI over it.
`internal/store` owns the `~/.renfe` directory (atomic 0600 JSON files);
`internal/config` is the `config.toml` schema on top of it.

Inside `internal/client`, one file per API call, each holding its own wire
struct and the projection onto the public types — a vertical slice, not a
layer.

- **client.go** — session, cookie jar, headers, retries, error model. The headers, `accept-language` included, are fixed: Renfe's beans reply in Spanish whatever they say.
- **transport.go** — uTLS Chrome TLS fingerprint.
- **dwr.go** — the DWR wire protocol.
- **search.go** — the journey search built on the three above.
- **stations.go** — the station catalogue and name matching.
- **account.go** — the signed-in beans (whoami, +Renfe card, trips).
- **browsercookies.go** — lifting a session out of a browser's cookie store.

In `cmd/renfe` the commands fetch and filter; they do not lay out output.
`print.go` holds the human-readable views (each takes an `io.Writer`, so a
layout is testable without capturing the process's stdout) and `output.go` the
`--json`/`--toon` emitters.

### The request sequence is stateful and ordered

Renfe keys the search context by HTTP session, not by request parameters:

1. `bootstrap()` — one-time `GET /vol/inicio.do` seeds the `JSESSIONID`; a search POST without one is rejected. Guarded by `sync.Once`.
2. `submitSearchForm()` — `POST /vol/buscarTren.do` stores the route **server-side**. Its HTML response is deliberately discarded: the page renders its train list client-side from the DWR call that follows.
3. `callDWR()` — the bean reads that stored context back out.

Consequence: one `Client` cannot search two different routes concurrently, and
the steps cannot be reordered. A round trip is **one** search, not two: when the
form carried a return date the single reply holds both directions in
`listadoTrenes`, keyed by `viajeIda`, and the return lands in `Results.Return`.

### DWR without a JavaScript engine

`POST /vol/dwr/call/plaincall/<bean>.<method>.dwr` answers with a *JS program*
that calls `r.handleCallback("0","0",<payload>)`. `decodeDWR` slices that
argument — objects and arrays by string-aware bracket matching, but also bare
strings and scalars, since `getIdentificacionUsuario` returns a plain string —
then `jsObjectToJSON` quotes the bare identifier keys, which is the only
difference from JSON. A reply missing the marker means an expired session or a
WAF bounce, not a parse bug.

`scriptSessionId` is DWR's CSRF token as well as its routing key: it must carry
the `DWRSESSIONID` cookie as its prefix or the server answers with a
`SecurityException`. A session that has never touched the DWR endpoint has no
such cookie and the check is waived — which is why the anonymous search path
works with a made-up id. See `scriptSessionID()`.

Request bodies are byte-reproducible on purpose: DWR declares each map value as
its own `c0-eN` variable and references it, `DWRParam` order mirrors the
browser's call order, and `dwrEscape` keeps spaces as `%20` (DWR's decoder does
not read `+` as a space). A zero-argument bean must omit `c0-param0` entirely.
Keep that fidelity when adding beans.

### Bot detection

`venta.renfe.com` scores the TLS fingerprint. `newChromeTransport()` presents
Chrome's JA3 via uTLS with ALPN pinned to `http/1.1` (so a plain
`http.Transport` carries it; JA3 hashes extension types, not the ALPN list). If
it fails to initialise, `New()` silently falls back to the stdlib transport and
warns once on the first request. Changing `DefaultUA` without the matching
`utls.HelloChrome_*` id breaks the disguise.

### Conventions

- **The library never touches os.Stderr/Stdout.** Diagnostics go through the optional `Client.Logf` hook; the caller routes them.
- **Errors:** non-2xx becomes `*APIError`; branch with `client.HTTPStatus(err)`, not type assertions. `doRetry` already retries 429/503 (3 attempts, `Retry-After` honoured, jittered backoff) — don't add a second retry layer above it.
- **Wire structs keep Renfe's Spanish field names** (`listviajeViewEnlaceBean`, `tarifaMinima`); the exported types are English. Unlisted fields are dropped by the decoder — the beans return far more than is consumed.
- Prices are comma-decimal strings (`"1.234,50"`) — always through `dwrPrice`.
- Availability is `!completo && price > 0`. The bean's `operativo` (a seat-map flag, false on plenty of bookable trains) and `razonNoDisponible` (an opaque code that rides along on available trains too) are deliberately not decoded — don't add them back.
- The station catalogue is ISO-8859-1; `latin1()` decodes it. Ranking ties break on Renfe's own `nmroPrioridad`, which is what makes `madrid` resolve to `MADRID (TODAS)`.
- CLI commands parse through `parseFlags`, never `fs.Parse` directly — `reorderArgs` is what lets a flag follow the positionals.

## Testing

`internal/client/testdata/*.dwr` are real captured replies (a one-way
Madrid→Barcelona search and a round trip); they cover the decoder and the
projection without touching the network. No test in the repo makes a live
request.

---
> Source: [seifreed/renfecli](https://github.com/seifreed/renfecli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
