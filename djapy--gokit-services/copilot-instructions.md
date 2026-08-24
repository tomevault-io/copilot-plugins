## gokit-services

> Manages the lifecycle of multiple services. Shutdown is triggered by: SIGINT/SIGTERM, ctx cancellation, a service error, or `ep.Shutdown()`.

# gokit-services

Reusable Go toolkit for microservices. Go 1.26. Module: `github.com/DjaPy/gokit-services`.

## Structure

All library code lives under `pkg/`; `example/` and `docs/` sit alongside it at the repo root.

```
pkg/
  core/
    entrypoint/   — application lifecycle
    service/      — Service, Shutdown, Prober interfaces
  http/
    server/       — HTTP server with Prometheus and panic recovery (package server)
    client/       — HTTP client with a middleware chain (package client)
  grpc/
    server/       — gRPC server (package server)
    client/       — gRPC client (package client)
  kafka/          — dialer/TLS/SASL + producer/ and consumer/ subpackages
  healthserver/ periodic/ workerpool/ dbservice/ redisservice/  — infra services
  internal/       — shared helpers (retry, prom), not importable by consumers
example/          — runnable orders-service wiring every primitive
docs/             — SPDD analysis/prompt artifacts
```

Transports are grouped by protocol, and the contract/orchestration layer is moved
into `pkg/core/`. The `server`/`client` subpackages are best imported with aliases
(`httpsrv`, `httpcli`, `grpcsrv`, `grpccli`) — this avoids collisions of the generic
names and the clash with stdlib `net/http`. The infrastructure services (`healthserver`,
`periodic`, `workerpool`, `dbservice`, `redisservice`) remain leaf packages under `pkg/`.

## Interfaces (pkg/core/service)

```go
type Service interface {
    Start(ctx context.Context) error  // blocks until stopped; ctx is canceled on shutdown
}

type Shutdown interface {
    Stop(ctx context.Context) error  // optional graceful stop
}
```

## entrypoint

Manages the lifecycle of multiple services. Shutdown is triggered by: SIGINT/SIGTERM, ctx cancellation, a service error, or `ep.Shutdown()`.

Order: PreStart hooks → Start (concurrently) → PostStart hooks → wait → PreStop hooks → Stop (concurrently) → PostStop hooks.

```go
import "github.com/DjaPy/gokit-services/pkg/core/entrypoint"

ep := entrypoint.New(
    entrypoint.WithServices(httpSrv, grpcSrv),
    entrypoint.WithShutdownTimeout(30 * time.Second),
    entrypoint.WithPreStart(func(ctx context.Context) error { ... }),
)
ep.Run(ctx)
```

## http/server

The HTTP server implements `service.Service` and `service.Shutdown`. It automatically collects Prometheus metrics and recovers from panics.

**Metrics:** `http_request_duration_seconds`, `http_response_size_bytes`, `http_requests_inflight`, `http_panic_recovery_total`. Labels: `http_service`, `http_handler`, `http_method`, `http_code`.

**Important:** Always pass `WithPrometheusRegisterer(prometheus.NewRegistry())` in tests — otherwise a second `NewServer` with the default registerer panics on duplicate metric registration. In production, use a single `NewServer` per process or your own `Registerer`.

```go
import httpsrv "github.com/DjaPy/gokit-services/pkg/http/server"

mux := http.NewServeMux()
mux.HandleFunc("GET /health", healthHandler)

srv := httpsrv.NewServer(mux,
    httpsrv.WithPort(8080),
    httpsrv.WithAppName("my-svc"),
)
```

A panic in a handler returns an RFC 7807 Problem JSON (`application/problem+json`) with status 500 to the client — only if the response hasn't started being sent yet.

`responseWriter` forwards `http.Flusher` and `http.Hijacker` — SSE and WebSocket work correctly.

## http/client

HTTP client with a fixed base URL and a middleware chain. The generic `Do[T]` decodes a JSON response into T.

```go
import httpcli "github.com/DjaPy/gokit-services/pkg/http/client"

c, err := httpcli.New("https://api.example.com",
    httpcli.WithTimeout(10 * time.Second),
    httpcli.WithMiddleware(authMiddleware, tracingMiddleware),
)

type User struct { Name string `json:"name"` }
user, err := httpcli.Do[User](ctx, c, http.MethodGet, "/users/42")
```

**`Do[T]`** returns an error on non-2xx. For an empty body (204 No Content) it returns the zero value without an error.

**Middleware:** the first in the list is the outermost (runs first). It is applied after all Options, so `WithMiddleware` and `WithTransport` can be passed in any order.

**`WithBody`** sets `Content-Length` automatically for types implementing `Len() int` (`*bytes.Buffer`, `*strings.Reader`).

## grpc/server, grpc/client

Both collect Prometheus metrics out of the box through built-in unary and stream interceptors,
prepended (outermost) so any interceptors added via `WithServerOptions` / `WithDialOptions` are
still timed and counted. Configure with `WithAppName` and `WithPrometheusRegisterer` (as with
`http/server`, pass `prometheus.NewRegistry()` in tests — `internal/prom.RegisterOrReuse` also
makes repeated construction on the default registerer safe).

**Metrics:** `grpc_server_handled_total`, `grpc_server_handling_seconds`,
`grpc_server_requests_inflight` (server) and the matching `grpc_client_*` (client). Labels:
`grpc_service` (app name), `grpc_type` (`unary` / `client_stream` / `server_stream` /
`bidi_stream`), `grpc_method` (full RPC method), `grpc_code` (status code; inflight gauge omits
it). Server-stream duration is the handler's lifetime; client streams are observed on
termination (the wrapping `ClientStream` records when the final `RecvMsg` returns).

## Patterns

- **Functional options** everywhere: `Option func(*T)` for the constructor, `RequestOption func(*http.Request)` for requests.
- **slog** for logging — `slog.Default()` by default, overridable via `WithLogger`.
- **Prometheus** — isolated registerers for tests, the default one for production.
- Tests use `httptest.NewServer` / `httptest.NewRecorder`, real HTTP connections (no transport mocks).

## Commands

```bash
go test ./...          # all tests
go test -race ./...    # with the race detector
```

## Versioning and releases

The module is a pure library (no `cmd/`), so the version is set exclusively by git tags;
no build-time version injection (`-ldflags`) is needed. Consumers pin the version via
`go get github.com/DjaPy/gokit-services@vX.Y.Z`.

**Policy (SemVer, `vMAJOR.MINOR.PATCH`):**
- While the project is in `v0.x.y` (no first release yet): `MINOR` — new features and breaking
  changes, `PATCH` — bugfixes only, no API changes. Backward compatibility between `0.x` releases
  is not guaranteed — that is expected for pre-1.0 per the spirit of SemVer.
- After the first `v1.0.0`: `PATCH` — bugfixes, `MINOR` — backward-compatible features,
  `MAJOR` — breaking changes.
- **Moving to `v2.0.0`+**: the module path must gain a `/v2` suffix (`module
  github.com/DjaPy/gokit-services/v2` in `go.mod`) — this is a Go modules requirement, not an option.

**Release process** (via `justfile`, git tags are the single source of the version, there is no `.version` file):
1. Update `CHANGELOG.md`: move the contents of `[Unreleased]` under a new heading
   `[X.Y.Z] - YYYY-MM-DD`, leaving `[Unreleased]` empty for the next changes; commit
   (`git commit -m "chore: release vX.Y.Z"`) and push to `main` — justfile does not automate this step.
2. Run one of the recipes (runs `all-check`: build+lint+test-coverage, then tags and pushes):
   ```bash
   just release-patch   # X.Y.Z -> X.Y.(Z+1) — bugfixes
   just release-minor   # X.Y.Z -> X.(Y+1).0 — new features / breaking changes before v1.0.0
   just release-major   # X.Y.Z -> (X+1).0.0 — breaking changes after v1.0.0
   ```
   The current version is read from `git describe --tags --abbrev=0` — a new tag is always counted
   from the latest existing one, nothing is entered manually. For a tag without a push, use
   `just bump-patch` etc. (the tag is created locally; push it separately with `git push --tags`).
3. `.github/workflows/release.yml` reacts to a `v*` tag push: it runs CI (reusing `ci.yml` as a
   reusable workflow — that one declares `workflow_call`) and creates a GitHub Release with
   auto-generated notes — nothing needs to be created manually.

**Breaking changes before `v1.0.0`**: allowed in `MINOR` releases, but must be explicitly marked
in `CHANGELOG.md` under a `### Changed` heading with a **BREAKING** note.

---
> Source: [DjaPy/gokit-services](https://github.com/DjaPy/gokit-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
