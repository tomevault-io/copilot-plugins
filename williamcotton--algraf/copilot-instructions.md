## algraf

> Guidance for working in the Algraf repository.

# CLAUDE.md

Guidance for working in the Algraf repository.

## What Algraf is

Algraf is a block-scoped, algebraic grammar-of-graphics DSL (file extension
`.ag`). It parses a declarative chart description, validates it against CSV
data, trains scales, and emits deterministic SVG. The whole system — parser,
language server, runtime, and renderer — ships as one Rust binary named
`algraf`.

## General DSL and API approach

Keep Algraf's language surface declarative and visualization-focused. The DSL
may name data sources, columns, stats, scales, guides, marks, and rendering
options; it must not grow awareness of producer languages such as PDL, concrete
data engines such as Polars, or storage details such as Arrow arrays and
Parquet row groups.

Use strict crate boundaries when adding performance work:

- `algraf-data` owns concrete loaders and conversion details for native formats.
- `algraf-driver` owns caller-data source resolution, format selection,
  sniffing, reader construction, and unsupported-format diagnostics.
- `algraf-render` owns visual semantics and consumes `Table`, typed column
  views, scan helpers, handles, or aggregate APIs. It must not import Arrow,
  Parquet, Polars, SQLite, or other native data-engine symbols.
- Parser, syntax, semantics, editor services, ordinary LSP surfaces, and public
  browser/WASM APIs must stay independent of concrete native data engines.

Native performance features are CLI/native concerns unless a plan explicitly
says otherwise. Do not enable Arrow stream, Parquet, Polars, SQLite, native
filesystem, or similar data-engine support in `algraf-wasm`; browser callers
should receive stable unsupported-format diagnostics for native-only formats.

## Spec and versioned plans

Three artifacts govern behavior, and they must stay in sync:

1. **`docs/ALGRAF_SPEC.md` — the normative reference.** It describes what the
   implementation *does*, not just what it might do. **Read the relevant section
   before implementing or changing behavior.** The spec uses RFC-2119-style
   keywords (`MUST`, `SHOULD`, `MAY`, `MUST NOT`) and assigns stable diagnostic
   codes (e.g. `E0012`); honor both. When code intentionally deviates from a
   `SHOULD`, document why in a comment.

2. **`docs/V0_<minor>_PLAN.md` — per-release planning files.** Each release gets
   one (e.g. `V0_2_PLAN.md`, `V0_3_PLAN.md`). A plan states the release thesis,
   lists Must/Should items with a `Status:` line each, and records what stays
   deferred. Plans are *guidance*, not normative: a feature is only real once the
   spec says `MUST`/`SHOULD` and the code implements it. The earliest
   unreleased plan is the active implementation target; later unreleased plans
   are sequencing guidance, and completed release plans are historical records —
   don't reopen a completed release's scope.

3. **The code** under `crates/`, plus its tests and `examples/`.

### How they tie together

- **Promoting a deferred feature** (an old `MAY`, "later versions", or "deferred"
  item) into a release: add it to the active plan file, then move it into the
  relevant normative spec section (`MUST`/`SHOULD`), reserve any new diagnostic
  codes in the spec *before* implementing, then implement + test + add an example.
  See the Promotion Workflow at the bottom of each plan file.
- **The spec must match the implementation, not run ahead of it.** If you ship a
  geometry, property, theme, scale/guide key, CLI flag, or diagnostic, document
  it in the spec in the same change. If you defer something, the spec must say so
  (`MAY`, "deferred") rather than describe it as though it exists. Diagnostic
  codes emitted by code must exist in the spec; the spec MAY reserve codes the
  code does not yet emit.
- **Keep the language-reference templates in sync.**
  `crates/algraf-cli/templates/ALGRAF_LANG.md` is the full agent-facing language
  reference that `algraf init` writes into a project, and it is a tracked
  language-surface artifact like the TextMate grammar and `README.md`. When the
  split language-reference templates exist, treat
  `crates/algraf-cli/templates/ALGRAF_LANGUAGE.md` as the source for Algraf
  syntax, declarations, algebra, geometries, properties, scales, guides, themes,
  enum values, data-source forms, and rendering semantics; treat
  `crates/algraf-cli/templates/ALGRAF_TOOLING.md` as the source for CLI
  commands, caller-data workflows, package/WASM APIs, agent setup, workflow
  guidance, and operational pitfalls. The full `ALGRAF_LANG.md` must be composed
  from those parts and must not drift. Whenever you add or change a geometry,
  geometry property, scale/guide key, theme token, chart argument, enum value,
  grammar shape, CLI flag/subcommand, WASM API, package API, or generated agent
  workflow, update the relevant template source section in the same change (for
  example language sections such as "Complete Declaration Reference", "Complete
  Geometry Property Reference", and "Property Value Forms And Enums", or tooling
  sections such as "CLI Commands" and "Project Agent Setup"). Like the spec, the
  templates document only the *implemented* surface — update them alongside
  implementation, never ahead of it, so a project on the current binary cannot be
  told about syntax or tooling that errors.
- **Keep plan examples runnable.** `.ag` snippets in plan files must use only
  features the implementation actually accepts (e.g. `Smooth(method: "lm")`, not
  `"loess"` while loess is deferred).
- **When a plan item lands, update its `Status:` line**; when a release ships,
  start the next `V0_<minor>_PLAN.md`.
- **Create the plan artifact even if work starts first.** Every feature,
  maintenance back-port, or release-scoped fix must have a current/new
  `docs/V0_<minor>_PLAN.md` entry. If implementation starts before planning is
  written down, create or update the appropriate plan artifact in the same
  change rather than editing an old completed plan. If the work is outside the
  active plan's declared purpose/scope, or if the active plan's items are
  already `Implemented`, start the next minor `docs/V0_<minor>_PLAN.md` instead
  of appending to the previous release. When that plan starts a new current
  release, immediately align the workspace/spec/package version stamps for that
  release (`Cargo.toml`, `Cargo.lock`, `docs/ALGRAF_SPEC.md`,
  `editors/vscode/package.json`, `editors/vscode/package-lock.json`, and any
  demo/package manifests that carry an Algraf release version). When the release
  changes implemented language, CLI, WASM, package, or agent workflow surface,
  update the language/tooling reference templates in the same change. Always
  update `docs/ALGRAF_SPEC.md` alongside the implementation.
- **When a plan is completed, update every release version before you stop.**
  This is not optional housekeeping. Bump the workspace version in `Cargo.toml`,
  refresh `Cargo.lock`, update `docs/ALGRAF_SPEC.md`'s draft status and
  milestone table, and align package manifests/lockfiles that track the release
  (`editors/vscode/package.json`, `editors/vscode/package-lock.json`, and any
  demo/package manifests that carry an Algraf release version). Confirm the
  language/tooling reference templates describe the implemented surface for that
  release. Do not mark a plan `Implemented` while leaving the repository stamped
  with the previous release.

If you find the spec, a plan, and the code disagreeing, treat it as drift to
fix — reconcile all three rather than picking one.

### NPM package version checks

Algraf browser packages are not guaranteed to be published for every Rust/CLI
version bump. Treat local package version stamps and published npm dependency
pins as separate concerns.

Before changing any npm package `version` field or npm-consumer version for
internal Algraf packages, verify that the target version actually exists on npm:

```bash
npm view algraf-wasm versions --json
npm view algraf-editor versions --json
```

This applies to package manifests for `algraf-wasm` and `algraf-editor`, demo
dependencies, downstream install instructions, peer dependency ranges,
package-lock `node_modules/algraf-*` entries, and package docs. Do not point a
package version, consumer dependency, or install command at
`algraf-wasm@<version>` or `algraf-editor@<version>` unless npm confirms that
exact version exists, or the user explicitly says the change is preparing an
unpublished local package release. If a package is not published for the new
workspace version, keep package and consumer pins on the latest verified
published version and document that browser package publication is independent
from the Rust/CLI release.

## Workspace layout

Cargo workspace with ten crates under `crates/` (see spec §23):

| Crate              | Responsibility                                              |
| ------------------ | ----------------------------------------------------------- |
| `algraf-core`      | Shared primitives: `Span`, `Diagnostic`, `Severity`         |
| `algraf-syntax`    | Lexer, parser, AST/CST (rowan), parse diagnostics, formatter|
| `algraf-data`      | CSV loading, schema inference, dataframe, type inference     |
| `algraf-semantics` | Name resolution, validation, IR, geometry registry          |
| `algraf-driver`    | Shared source resolution, data/schema loading, analysis prep|
| `algraf-render`    | Scale training, layout, stats, geometries, SVG emission     |
| `algraf-editor-services` | Shared editor features: completion, hover, tokens, actions |
| `algraf-lsp`       | tower-lsp backend, document cache, LSP transport             |
| `algraf-cli`       | The `algraf` binary: arg parsing, command dispatch, I/O     |
| `algraf-wasm`      | Browser/WASM runtime: in-memory `DriverIo`, reuses driver→render|

Dependency direction flows downward: `core` depends on nothing internal;
`driver` depends on syntax, data, and semantics; `editor-services` depends on
syntax and semantics; `lsp` wraps editor-services; `cli` depends on the runtime
stack. Do not introduce cycles. Keep parser, LSP, semantics, and render
decoupled from concrete dataframe internals (spec §10.5).

## Building and running

Build the binary with `cargo build -p algraf-cli`; it lands at
`target/debug/algraf`. To try a chart while iterating:

```bash
cargo run -p algraf-cli -- render examples/scatter.ag --output /tmp/out.svg
cargo run -p algraf-cli -- check examples/scatter.ag      # parse + analyze, no render
```

Other subcommands: `format`, `schema`, `ast`, `ir`, and `lsp` (see spec §22).
`./examples/generate.sh` builds the CLI and re-renders every example.

## Required checks before finishing any change

Run all three from the repo root and make sure they pass:

```bash
cargo fmt --all          # format (CI uses `cargo fmt --all --check`)
cargo clippy --workspace --all-targets   # lint; treat warnings as failures
cargo test --workspace   # run the full test suite
```

A change is not done until `cargo fmt --all --check`, `cargo clippy
--workspace --all-targets`, and `cargo test --workspace` are all clean. Clippy
runs over tests too (`--all-targets`), so keep test code lint-clean as well.

## VS Code client

`editors/vscode/` is a thin VS Code language client (TypeScript, `src/extension.ts`,
bundled with esbuild). It does **not** reimplement any language logic: it spawns
the `algraf` binary as a language server (`algraf lsp` by default, configurable
via `algraf.server.path`/`algraf.server.args`) and talks LSP to it. All
completion, hover, diagnostics, semantic tokens, formatting, and code actions
come from the `algraf-lsp` crate — so LSP features are exercised by that crate's
tests, and the extension just wires them into the editor.

Practical implications:

- Improve editor behavior by changing `algraf-lsp`, not the extension. Touch the
  extension only for client wiring (config, commands, activation, packaging).
- Two things in the extension are local copies that must track the language:
  the TextMate grammar (`syntaxes/algraf.tmLanguage.json`) and
  `language-configuration.json`. If you add keywords, geometry names, or
  punctuation, update the grammar so static highlighting matches what the LSP
  reports via semantic tokens.
- Keep `package.json`'s `version` aligned with the workspace release version
  when cutting a release (it is currently versioned alongside the crates).

## Example Generation

Run `./examples/generate.sh` to regenerate the SVG and PNG outputs for all examples.

When you add or modify an example chart, or touch renderer behavior that changes
example output, inspect the changed rendered PNGs before finishing. Verify the
marks visually match the example's intent: positions align with the data, axes
and guides are sensible, legends match mapped aesthetics, labels do not overlap
unacceptably, and layered marks share the coordinate system they are meant to
share. Do not rely only on `algraf check`, SVG byte diffs, or unit tests for
chart examples.

Create new examples by adding a new file to the `examples/` directory.

Create new examples when adding new features or fixing bugs.

When you add a new example, also add a section for it to the top-level
`README.md` — the README is a tutorial that shows every example's `.ag`
source followed by its rendered SVG, so it must stay in sync with the
contents of `examples/`. Place the new section where it fits the tutorial
progression (basics → layering → stats → layouts → derived tables → annotations
→ theming), not just at the end.

## Conventions

- Diagnostics are values, not exceptions. Parser/analyzer/renderer return their
  output plus a `Vec<Diagnostic>`; reserve `panic!` for programmer bugs (spec
  §23.4).
- Every token and syntax node carries a byte-offset `Span` (spec §6.12, §11.2).
  Spans are half-open `[start, end)`. Always test byte offsets with non-ASCII
  input — they are bytes, not chars.
- The lexer/parser are resilient: recover and continue on bad input, emit a
  diagnostic, and never panic (spec §12.1, §27.4).
- Output must be deterministic — stable ordering, no time/locale dependence
  (spec §18.12, §23.6).
- Tests live in each crate's `tests/` directory and follow the categories in
  spec §27. Add tests alongside new behavior.

## Commits

Do not create git commits. Make file edits, run the required validation
checks, and stop. Every commit must be authored manually by a human
author after they review the working tree. Stage and commit only when
explicitly asked to do so for a specific commit, and never as part of
finishing a task or closing out a multi-step plan.

---
> Source: [williamcotton/algraf](https://github.com/williamcotton/algraf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
