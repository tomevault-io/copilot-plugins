## primeval

> `primeval` is a Rust workspace plus a small Node package surface.

# AGENTS.md

## Repo Shape

`primeval` is a Rust workspace plus a small Node package surface.

Main components:

- `crates/primeval-core`: optimization engine, rasterization, scoring, and export
- `crates/primeval-render`: high-level decode -> render -> encode facade shared by binding and package surfaces
- `binding`: napi-rs crate named `primeval-node`
- `src/index.ts`: single-source TypeScript wrapper for the npm package
- `src/cli.ts`: Node CLI entrypoint distributed via npm package `bin`
- `test/`: Node/package tests
- `docs/readme/`: README assets
- `docs/plans/`: active planning docs; keep this directory small and current

Generated output:

- `dist/` is build output for the npm package; do not edit it by hand

## Repo Direction

- The npm package is ESM-only and targets Node 20+.
- Browser/WASM support is explicitly out of scope.
- CommonJS support is explicitly out of scope.
- Rust is the runtime source of truth for accepted vocabularies, defaults, and validation semantics.
- The final Rust API is the only source of truth for render-option defaults. If a value is omitted, `undefined`, or intentionally left unset, wrapper and binding layers should pass that absence through so Rust can decide the default.
- TypeScript mirrors the Rust contract; do not introduce cross-language code generation or shared schema systems.
- Do not add new public render options unless required to make existing behavior consistent across render, binding, and TypeScript.

## Implementation Rules

- Keep `primeval-render`, `binding`, `src/index.ts`, and `src/cli.ts` aligned on defaults, accepted values, and error behavior.
- Do not reinvent Rust defaults in TypeScript or napi request shims. Validate explicit user input there, but let Rust merge omitted fields onto its own defaults.
- Prefer shared Rust parsers/helpers over duplicated string tables in binding.
- Keep the TypeScript wrapper thin. Canonical runtime behavior should live in Rust unless there is a strong package-layer reason not to.
- Keep `npm run build`, `npm run typecheck`, and `npm pack --dry-run` viable from a clean checkout before any local native build step. If runtime-generated native files are unavoidable, isolate them behind a small shim and cover the clean-checkout path with a tooling test.
- Red-green-refactor TDD is a hard rule for code changes: add or update a failing test first, make it pass with the smallest useful change, then clean up.
- During local development, prefer autofix variants of formatters and linters when they exist so routine cleanup is cheap for agents. Use check-only commands for final verification and CI.
- Avoid duplicated CI/release verification logic. Prefer one reusable quality path.
- Treat missing native artifacts as release failures even if packaging commands only warn.
- Pin CI tooling versions. Do not rely on floating-branch downloads or unpinned CLI installs in workflows.
- For native package targets, keep one canonical source of truth in `package.json` `napi.targets`; derive or validate release metadata and artifact checks from it.
- Treat `package-lock.json` as npm-managed output: do not hand-edit it, regenerate it with npm when package metadata changes, and verify the result with `npm ci`.
- If public package behavior changes, update `README.md` in the same change.
- If a plan is implemented or superseded, prune or reduce the corresponding file in `docs/plans/`.

## Build And Verification

Run commands from the repository root.

Core Rust checks:

```bash
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo test
```

Core Node/package checks:

```bash
npm ci
npm run typecheck
npm test
npm pack --dry-run
```

Useful build commands:

```bash
cargo build --release
npm run build
npm run build:node
```

## Benchmarking

Use `scripts/benchmark.py` to compare two binaries:

```bash
python3 scripts/benchmark.py \
  --bin-a ./dist/cli.js \
  --bin-b /path/to/other/binary \
  --label-a current \
  --label-b baseline \
  --no-build
```

Reports are written to `output/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aleburato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
