## commandcode-proxy-go

> Operational guidance for AI coding agents working in this repository.

# AGENTS.md

Operational guidance for AI coding agents working in this repository.
Humans: see [README.md](README.md) for the overview and install/usage, and
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full design rationale
(decisions referenced below as **D1–D16** live there).

## What this is

`commandcode-proxy` (Go) is a stateless, stdlib-only `net/http` service that
exposes an **OpenAI Chat Completions**–compatible API (`/v1/chat/completions`,
`/v1/models`, `/health`) plus an **Anthropic Messages**–compatible `/v1/messages`,
and translates every call to Command Code's custom `/alpha/generate` endpoint. It
ships as a single static binary. It is a port of the Python
[`commandcode-proxy`](https://github.com/liwei/commandcode-proxy).

## Layout

| Path | Responsibility |
|---|---|
| `cmd/commandcode-proxy/main.go` | Entrypoint: env-driven host/port, starts `http.Server` |
| `internal/server/server.go` | Routes, response assembly, error mapping, shared HTTP client |
| `internal/server/dashboard.go` | `/admin` dashboard + request-log middleware (metadata-only ring) |
| `internal/translate/translate.go` | **Pure** OpenAI ⇄ Command Code request build + shared mappers |
| `internal/translate/openai_response.go` | **Pure** `chat.completion` / chunk / SSE builders |
| `internal/translate/anthropic.go` | **Pure** Anthropic ⇄ Command Code; reuses `translate`'s unexported helpers |
| `internal/upstream/upstream.go` | `/alpha/generate` streaming, retry, error classification |
| `internal/auth/auth.go` | API-key extraction from `Authorization` / `x-api-key` (keyless) |
| `internal/config/config.go` | Environment-driven settings, model-alias resolution |

Dependency direction is one-way: `server → {auth, config, translate, upstream}`,
`upstream → {translate, config}`. Never make `translate`, `auth`, or `config`
import from the rest of the module. The Anthropic translation lives **inside** the
`translate` package (not a separate package) so it can reuse the unexported
Command Code helpers — the Go analog of the Python `translate_anthropic`
importing `translate`.

## Setup · run · test

```sh
# run (serves http://127.0.0.1:8787; keyless — callers pass their own key)
go run ./cmd/commandcode-proxy

make build      # -> ./commandcode-proxy (static, CGO disabled)
make test       # vet + unit + route tests — MUST stay green, no network
make release    # cross-compiled static binaries in ./dist

# live end-to-end — opt-in: run the binary, then curl it with a real key
# (key stays ephemeral in the shell/header — never put it in a tracked file)
curl http://127.0.0.1:8787/v1/chat/completions \
  -H "Authorization: Bearer user_..." -H "Content-Type: application/json" \
  -d '{"model":"Qwen/Qwen3.7-Plus","messages":[{"role":"user","content":"PONG"}]}'
```

## Invariants — do not break these

1. **`internal/translate` stays pure.** No network, no file/env I/O, no global
   mutable state — input maps in, output maps out. All I/O belongs in `upstream` /
   `server`. This keeps the risky translation logic unit-testable. (D4)
2. **Peek the first decisive event before committing a response.** Upstream errors
   must surface as a real HTTP status (`403`/`401`/…), never as a `200` stream that
   hides an error. See `upstream.AdvanceToDecisive` and `server.chatCompletions`. (D5)
3. **Retry only before the first content event.** Never retry after any
   `text-delta` / `reasoning-delta` / `tool-call` — it would duplicate output.
   Retryable cases: HTTP `429`/`5xx` at open, and an in-stream `error` flagged
   `isRetryable`. (D6)
4. **The public contracts are OpenAI Chat Completions and Anthropic Messages.**
   Don't change request/response wire shapes casually; consumers depend on them.
   Build outputs as `map[string]any` so null/omit/empty nuances stay exact. (D16)
5. **Forward to `/alpha/generate`, not `/provider/v1`.** The latter requires a paid
   Provider tier — defeating the proxy's purpose. (D3)
6. **Streaming and buffered responses stay consistent** — both consume the same
   event stream through the shared `translate` helpers. (D8)
7. **Never log or echo the API key.** Errors carry Command Code's message text
   only; the dashboard records metadata only (no key, no content). (D7, D15)
8. **Dashboard middleware must not buffer streams.** The `ResponseWriter` wrapper
   delegates `http.Flusher`, so SSE keeps flushing. Don't add a wrapper that
   swallows `Flush`. (D15)

## Conventions

- Go 1.23+. Prefer the standard library, but adding a third-party dependency is
  fine when it earns its place (a clear need stdlib serves poorly). Zero
  dependencies is not a goal — just keep the set small and justified.
- `gofmt`-clean and `go vet`-clean before every commit (`make test` runs vet).
- Match the existing style: small focused functions; comments explain *why*, not
  *what*; comment only where non-obvious.
- Keep changes surgical — every changed line should trace to the task.

## Verifying changes

- `make test` must pass; add/adjust a test for any new translation or route
  behavior. Route tests drive a `httptest` fake Command Code backend over real
  HTTP, exercising `upstream` + `translate` end to end — prefer that to mocking.
- Treat plan-gated `402`/`403` (`MODEL_NOT_IN_PLAN`) from the live API as expected
  — model availability is per Command Code plan, not a proxy bug.
- Don't commit without review.

---
> Source: [liwei/commandcode-proxy-go](https://github.com/liwei/commandcode-proxy-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
