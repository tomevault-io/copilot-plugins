## kotlin-dsv

> Tooling is managed by `mise`. Run `mise install` first (installs Java, dprint, hk, downloads ktfmt

# CLAUDE.md

## Dev workflow

Tooling is managed by `mise`. Run `mise install` first (installs Java, dprint, hk, downloads ktfmt
jar, and sets up git hooks).

Key tasks:

- `mise run build` — compile and run all checks across all platforms
- `mise run test` — all tests (JVM, JS, WASM, native)
- `mise run test:jvm` / `test:jsnode` / `test:wasmjsnode` / `test:native` — individual platforms
- `mise run check` — lint and format checks (hk)
- `mise run fix` — auto-fix formatting

Run a single test: `./gradlew :module:jvmTest --tests "*SomeTest*"`

## Pitfalls

**After changing any public API**, run `mise run fix` to regenerate the `.api` files — the build
fails without this.

**Floating-point comparisons in tests**: use helpers from `testutil` instead of `assertEquals` to
handle platform-specific precision differences (if applicable).

---
> Source: [sargunv/kotlin-dsv](https://github.com/sargunv/kotlin-dsv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
