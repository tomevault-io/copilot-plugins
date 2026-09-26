## freehire

> Run gofmt, go vet, and unit tests before committing Go files


# Before committing Go

When creating a git commit that includes any `*.go` file:

1. `gofmt -w` those paths. `gofmt -l .` must print nothing.
2. `go vet ./...`
3. `go test ./...` (unit tests only — no `-tags=integration`).

Do **not** commit if vet or tests fail; fix first.

Skip this suite when the commit has no Go files (docs, OpenSpec, web-only).

Do **not** run `go test -tags=integration ./...` at commit time. That stays push-time: `go vet -tags=integration ./...` before every push; the full tagged suite when behaviour changed.

---
> Source: [strelov1/freehire](https://github.com/strelov1/freehire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
