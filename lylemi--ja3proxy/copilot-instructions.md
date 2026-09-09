## ja3proxy

> JA3Proxy is a Go CLI application. The command entrypoint lives in

# Repository Guidelines

## Project Structure & Module Organization

JA3Proxy is a Go CLI application. The command entrypoint lives in
`cmd/ja3proxy/`, while runtime implementation and tests live in
`internal/ja3proxy/`. The root package handles CLI/runtime wiring; focused
subpackages cover `proxy/`, `tunnel/`, `fingerprint/`, `upstreamtls/`,
`certstore/`, `traffic/`, `dialer/`, `pipe/`, `tui/`, `netutil/`, and `e2e/`.
The module definition is in `go.mod` and `go.sum`. Build and container assets
are at the repository root: `makefile`, `Dockerfile`, and `compose.yaml`.
Static branding assets live in `assets/`.

## Build, Test, and Development Commands

- `go mod download`: fetch module dependencies.
- `go build -v ./...`: compile all packages, matching CI behavior.
- `go test -v ./...`: run the full test suite.
- `go build -o ja3proxy ./cmd/ja3proxy`: build a local executable.
- `make` or `make all`: build Linux and Windows amd64 binaries into `bin/`.
- `make clean`: remove Makefile-generated binaries.
- `docker compose up -d`: run the proxy with the Compose example on port 8080.

Run locally with, for example:

```bash
./ja3proxy --listen :8080 --tls-fingerprint Chrome@106
```

## Coding Style & Naming Conventions

Use standard Go formatting. Run `gofmt` on edited Go files before committing,
and keep imports organized by `go fmt`/`goimports` conventions. Prefer small,
focused files aligned with existing protocol boundaries. Use idiomatic Go names:
export only symbols needed outside a file/package, keep local helpers in
lowerCamelCase, and name tests `TestFeatureOrBehavior`.

## Testing Guidelines

Tests use Go's standard `testing` package and are colocated with source files as
`*_test.go`. Add targeted tests when changing proxy routing, SOCKS5 parsing,
TLS fingerprint selection, certificate handling, runtime lifecycle, or config
reload behavior. Run `go test -v ./...` before opening a pull request.

## Commit & Pull Request Guidelines

Use concise, imperative Conventional Commit messages. Prefer the scoped pattern
`<type>(<scope>): <summary>`, such as `feat(proxy): add upstream TLS profile
routes` or `test(socks5): cover request parsing edge cases`. Use
`chore(deps): ...` for dependency updates. Keep the subject specific, under
roughly 72 characters when practical, and keep each commit focused on one
logical change. Pull requests should describe the behavior change, list
validation performed, link related issues when available, and include logs or
curl examples for proxy behavior changes.

## Security & Configuration Tips

Do not commit generated CA material or local runtime secrets. The repository
ignores `credentials/`, `*.pem`, `bin/`, and local `ja3proxy` binaries. When
testing HTTPS interception, use disposable local certificates and document any
required client trust setup in the PR.

---
> Source: [LyleMi/ja3proxy](https://github.com/LyleMi/ja3proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
