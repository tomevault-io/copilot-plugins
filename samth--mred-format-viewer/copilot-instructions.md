## mred-format-viewer

> Terminal viewer for DrRacket's WXME binary file format, built with the Kettle TUI library.

# CLAUDE.md

## Project

Terminal viewer for DrRacket's WXME binary file format, built with the Kettle TUI library.

## Language & Dependencies

Racket. Runtime deps: `wxme-lib`, `kettle-lib`. Test deps: `rackunit-lib`, `kettle-test-lib`.

## Running

```sh
racket -y main.rkt -- <file>
```

## Testing

```sh
raco test -y tests/
```

Tests use both direct model unit testing and the `kettle/test` headless harness for e2e tests. The `tests/examples/` directory contains real WXME files (renamed to `.wxme` so `raco test` doesn't try to compile them).

## Architecture

Single-module design in `main.rkt`:

- **Data structures**: `snip-info`, `line-ann`, `wxme-viewer` (implements `gen:kettle-model`)
- **WXME reading**: `make-wxme-viewer` opens a file, decodes via `wxme-port->port` with a snip-filter, consumes the decoded port into an elements list
- **Display**: `elements->display` flattens elements into line/annotation vectors; detail lines inserted for expanded snips
- **TUI**: Follows the `log-viewer.rkt` pattern from Kettle examples — manual scrolling with vectors, no viewport component, because we need per-line annotation metadata

The `wxme-viewer` struct and helpers are `provide`d for testability.

## Key conventions

- `(except-in kettle/program init)` is required because `init` conflicts with `racket/class`
- Use `(parameterize ([unknown-extensions-skip-enabled #t]) ...)` around all WXME decoding to handle missing extension snip classes gracefully
- Example WXME files use `.wxme` extension (not `.rkt`) to prevent `raco test` from trying to compile them
- Use `racket -y` to ensure recompilation

## CI

GitHub Actions with `Bogdanp/setup-racket`. The `wxme-lib` package comes pre-installed with Racket; only `kettle-lib` and `kettle-test-lib` need explicit installation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samth)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/samth)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
