## fort

> > The authoritative directives live in [`CLAUDE.md`](./CLAUDE.md) and apply

# Fort — Codex / Agent Directives

> The authoritative directives live in [`CLAUDE.md`](./CLAUDE.md) and apply
> equally to Codex and any other coding agent. Read it. This file restates the
> load-bearing rules so they're not missed.

## What Fort is
Deterministic agent orchestration in **Go**: route a task by fixed rules (no
model in the routing path), run it by spawning agent CLIs natively, sequence
multi-step work as a DAG that pauses at human gates. One binary; a control plane
for web / iOS / macOS / CarPlay / watch. The earlier TypeScript prototype was an
experiment and has been removed (git history preserves it). Governing spec:
`specs/021-fort-native.md`.

## Non-negotiables
- **Test-first (TDD).** Write the failing `go test` first, watch it fail, then
  the minimal code to pass. Keep `go test ./...` green; `-race` for concurrency.
- **Determinism is asserted.** Zero model calls in the routing path; only `task`
  DAG nodes invoke the `Runtime`. Preserve those invariants with tests.
- **Respect the seams.** `core` must not import `ui` or a concrete `exec`
  package (only `runtime.Runtime`); `ui` must not import
  `engine`/`graph`/`router`/`native`. Enforced by `core/arch_test.go` +
  `go list -deps`.
- **Fort owns the interface.** Reach an industry CLI/library only through a
  bounded, testable Fort contract (e.g. a `NativeRuntime` provider). New
  capability specs require Toby's approval before implementation.
- **Spec-driven, surgical, simple.** Spec in `specs/` first; touch only what the
  task requires; minimum code that solves it. See CLAUDE.md's behavioral
  guidelines.

## Layout
`core/` (rules, router, runtime iface, store, engine, graph, inbox, flow,
scheduler, server) · `exec/` (native, fake, gateway) · `ui/` (control-plane
HTTP/SSE + web) · `control/` (port adapters) · `cmd/fort/` (CLI) · `rules/` +
`flows/` (YAML) · `ui/apple/` (FortKit + Apple clients).

---
> Source: [tobsai/fort](https://github.com/tobsai/fort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
