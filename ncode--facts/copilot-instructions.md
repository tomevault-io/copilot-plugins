## facts

> Facts is a Go port of Puppet Facter. Use the project language in

# Repository Instructions

Facts is a Go port of Puppet Facter. Use the project language in
`CONTEXT.md`: the project and library are **Facts**, the CLI is `facts`, and
Ruby Facter compatibility is promised only at the CLI process boundary and
for operator-supplied fact sources.

## Development

- Requires Go 1.26 or newer.
- Run `gofmt -w` on edited Go files.
- Use `go test ./...` for the default test pass.
- Use `go vet ./...` before handing off changes.
- Use `go test -race . ./internal/engine ./internal/app` for concurrency-sensitive changes.
- Use `make build` to build the `./facts` CLI.

## Change Rules

- Behavior changes go through OpenSpec under `openspec/changes/`.
- User-visible behavior changes also update `CHANGELOG.md`.
- New or renamed facts update `docs/schema/facts.yaml`.
- Fact resolution and formatting changes need tests against the public surface: the root `facts` package API or `internal/app` CLI contract tests.
- Platform logic should be tested through fixtures or injected probes, not host-specific assumptions.

## Compatibility

- Preserve the output contract and input contract unless an ADR records the break.
- Ruby custom fact `.rb` files are not supported; external facts are the input contract.
- The canonical structured tree is the only fact surface; do not add legacy alias facts.
- Release targets are Linux, macOS/Darwin, Windows, and FreeBSD.

---
> Source: [ncode/facts](https://github.com/ncode/facts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
