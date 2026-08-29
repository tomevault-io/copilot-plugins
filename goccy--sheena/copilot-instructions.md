## sheena

> Guidance for AI agents and human contributors working in this repository. These

# AGENTS.md

Guidance for AI agents and human contributors working in this repository. These
rules are mandatory for every change. (A Claude-specific superset lives in
`CLAUDE.md`; this file is the tool-neutral version.)

## What Sheena is

A lightweight, in-process sandbox for safely executing the tool calls an AI
agent requests, on the same host as the agent. See `README.md` for the API and
`DESIGN.md` for the full design.

## Go version & idioms

- Target **Go 1.25.0**. Any feature up to and including Go 1.25 may be used
  (generics, `os.Root`, `io/fs.ReadLinkFS`, range-over-func, etc.).
- Prefer modern idioms. Use **`any`**, never `interface{}`. Use generics where
  they remove duplication or improve type safety.
- Keep `go.mod`'s `go 1.25.0` directive and the golangci-lint config in sync.

## Testing & coverage

- Every package MUST maintain **≥90% statement coverage**. Add tests until the
  bar is met; never lower it.
- `make test` (`go test -race ./...`, with the wasm interpreters run separately
  without `-race`) and `make cover` (per-package ≥90% gate) MUST pass.
- Access-control behavior is documented and asserted with **example tests**
  (`Example…` + `// Output:`) — path traversal blocked, network denied by
  default, overlay never writes to the host.
- Keep the dedicated suites: security/threat-model (`security_test.go`), escape
  attempts (traversal, symlink escape, SSRF, redirect, output/command/memory
  caps, timeout), and the differential golden tests that check commands against
  real coreutils (`internal/difftest`).

## Benchmarks

Sheena claims to be lightweight; that claim stays measured. Keep `Benchmark…`
for sandbox construction/mass startup (report `allocs/op` and `B/op`), per-call
`ExecBash` overhead, and parallel throughput (`b.RunParallel`). Run with
`make bench`; compare with `benchstat` when touching hot paths and do not
regress startup or allocations without a stated reason.

## Linting

`golangci-lint` MUST pass with **zero issues** (`make lint`). Fix the underlying
issue rather than disabling a linter or adding a `nolint` directive; if a
suppression is truly warranted, justify it in the diff.

## Code organization

- All filesystem code lives in the **`fs`** package.
- All commands live under **`sh/`**, **one file per command**, each with its own
  `*_test.go`. Core, stdlib-only commands sit in the `sh` package and are listed
  in `sh.Core()`. A command backed by a heavy dependency lives in its **own
  sub-package** (`sh/awk`, `sh/jq`, `sh/yq`, `sh/sqlite`, `sh/htmlmd`,
  `sh/python`, `sh/js`) so that dependency is compiled and linked only when
  referenced. `TestDependencyIsolation` enforces this — the root and `sh`
  packages must not pull in any heavy dependency.
- Commands are **value-exported** (`var Cat sh.Command = catCmd{}`) so unused
  ones are removed by dead-code elimination.
- The public API stays **adapter-free**: usable with any Go AI SDK via
  `Sandbox.Tools()` + `Sandbox.CallTool()`. Do not add a required per-SDK
  adapter layer.
- Prefer **cgo-free** dependencies (e.g. `modernc.org/sqlite`, the wasm
  interpreters).
- Language layer terminology: a **"tool"** is the LLM-facing surface
  (`Tools()`/`CallTool()`); a **"command"** is the shell-layer executable.

## Terminology & vocabulary

- The filesystem and network policy packages are deliberately named `fs` and
  `net` (mirroring the stdlib); this is intentional, not a mistake to "fix".
- Builders use method chaining (`fs.Mem().Seed(...).Refuse(...)`,
  `net.Allow(host).Port(443)`), not functional options.

## Dev tool management (`tools.mod`)

Development-only tools (golangci-lint, gofumpt) are recorded in **`tools.mod`**,
not the root `go.mod`. Add one with
`go get -tool -modfile=tools.mod <module>@<version>`; run one with
`go tool -modfile=tools.mod <name> …`; run `make tidy` after dependency changes.
Keep runtime deps out of `tools.mod` and dev tools out of `go.mod`.

## Examples module

`examples/agentframework` is its **own Go module** (with a `replace` back to the
root) so an AI-SDK dependency never enters Sheena's own dependency graph. It is
built and tested by `make example` and a dedicated CI job.

## Definition of done

A change is complete only when `make lint`, `make test`, and `make cover` all
pass locally. Do not report completion while any of them fails, and never leave
a known runtime defect (panic, data race, wrong-answer) unfixed.

---
> Source: [goccy/sheena](https://github.com/goccy/sheena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
