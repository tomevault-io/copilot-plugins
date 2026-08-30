## api-parity

> handles genuinely ambiguous cases. Produces the same envelopes

# api-parity (project context)

Cross-language API parity tracker. Public-facing intro lives in
[`README.md`](README.md); this file is for AI / agent context.

## What it is

Three packages, each released independently:

- **`api-parity/`** — language-agnostic differ + report renderer
  (CLI: `api-parity compare`). Left-joins two JSON inventories on
  `path`. Distributed as the `api-parity` PyPI package.
- **`api-parity-py/`** — Python plugin. Walker
  (`inspect.getmembers` over a package) + decorators (`@parity`,
  `@parity_impl`, `@parity_ref`). Distributed as `api-parity-py` on
  PyPI.
- **`api-parity-rs/`** — Rust plugin workspace. Two crates:
  `api-parity-rs` (runtime types + serde-gated `dump_to_writer` +
  walker behind a `walker` feature + CLI bin behind the default
  `cli` feature) and `api-parity-rs-macros` (`#[parity]` /
  `#[parity_impl]` proc-macros, kept separate because
  `proc-macro = true` crates can't ship anything else). Both
  crates released to crates.io in lockstep.

## Two orthogonal axes

`kind` is consumer-side, `mode` is producer-side:

|                    | `mode = walker` | `mode = annotation` | `mode = ast` |
| ------------------ | --------------- | ------------------- | ------------ |
| `kind = reference` | **Default.** Public-API walk. | `@parity_ref` decorators. | Parse source, no import. py only. |
| `kind = port`      | Walked surface as implemented. | **Default.** `@parity` / `#[parity]`. | — |

All four (kind, walker/annotation) combos work on both plugins as of
0.0.2. `ast` is `api-parity-py` only, reference only, as of py-0.0.3.

`walker` and `annotation` both load the target; `ast` doesn't, which is
the entire point of it — it exists for targets that can't be loaded at
all (a test suite a distribution doesn't ship) or can't be loaded here
(an import that drags in a dependency graph unrelated to the names being
read). The cost is that only lexically-present names are visible:
inherited members stay with the class that declares them, import-time
construction is invisible, and `kind` follows decorator spelling. It is
a fallback, not a default.

`target` stays the dotted name in every mode; `ast` is selected by
`--from-source PATH`, which says where to read it from instead of
importing it. Module names come from `target`, so a vendored tree keys
to the names it has upstream rather than to its checkout location.

## Wire format

[`SCHEMA.md`](SCHEMA.md) is canonical. Summary:

- Both sides emit
  `{schema_version: 1, kind, language, version, source, entries: [...]}`.
- Reference entries: `{path, kind}` with `kind ∈ {class, method, property, function}`.
- Port entries: `{path, implementation, status, since, issue, comment}`
  with `status ∈ {implemented, partial, unimplemented}`.
- `comment` is port-only and required when `status == unimplemented`.
- Plugin CLI: `api-parity-<plugin> <kind> [--mode …] <target> [-o PATH | -]`.
  Plugins exit 64 with a stderr message on `(kind, mode)` combos that
  aren't yet implemented.

## Status

Shipped: `api-parity` 0.0.3, `api-parity-rs` 0.0.3, `api-parity-py`
0.0.3. All four directions (py↔py, py↔rs, rs↔py, rs↔rs) work
end-to-end. See [`CHANGELOG.md`](CHANGELOG.md).

## Roadmap

**`api-parity-ai` plugin.** Natural-language-driven discovery and,
eventually, translation between path schemes (e.g. Rust `::` ↔
Python `.`). Translation should run a heuristic pass first
(substring / casing / module-prefix rewriting), so the model only
handles genuinely ambiguous cases. Produces the same envelopes
everything else does.

## Layout

```text
api-parity/                              # working directory (repo root)
├── CHANGELOG.md
├── CLAUDE.md                            # you are here
├── CONTRIBUTING.md                      # plugin authorship + release process
├── README.md                            # public-facing intro
├── SCHEMA.md                            # wire-format spec
├── Makefile                             # dev-shortcut targets
├── .github/workflows/                   # ci, build_test_draft (tag-driven), release
├── api-parity/
│   ├── pyproject.toml
│   ├── src/api_parity/
│   │   ├── __main__.py                  # CLI: `api-parity compare`
│   │   ├── compare.py                   # left-join + totals
│   │   ├── render_markdown.py
│   │   └── render_json.py
│   └── tests/                           # unit + CLI acceptance
├── api-parity-py/
│   ├── pyproject.toml
│   ├── src/api_parity_py/
│   │   ├── __main__.py                  # CLI: `api-parity-py {reference|port}`
│   │   ├── walk.py                      # introspection walker
│   │   └── parity.py                    # @parity / @parity_impl / @parity_ref
│   └── tests/                           # unit + CLI acceptance + fixture pkgs
└── api-parity-rs/
    ├── Cargo.toml                       # workspace
    ├── README.md                        # consumer guide
    ├── crates/
    │   ├── api-parity-rs/               # lib + CLI bin + walker (feature-gated)
    │   └── api-parity-rs-macros/        # #[parity], #[parity_impl]
    └── tests/fixtures/                  # portcrate, refcrate — fixture crates for acceptance tests
```

## Origin / what was ported

Code was lifted (and renamed) from `../spark-connect` on
2026-05-06, then renamed `parity` → `api-parity` on 2026-05-07.
The originals remain the closest thing to a reference
implementation — if you spot behaviors there that aren't here,
that's probably an oversight worth fixing.

| Original (in `../spark-connect`) | Now |
| -------------------------------- | --- |
| `tools/pyspark_inventory.py` | `api-parity-py/src/api_parity_py/walk.py` (flat `entries[]`) |
| `tools/parity_report.py` | `api-parity/src/api_parity/{compare,render_markdown,render_json}.py` |
| `crates/api-parity/src/lib.rs` | `api-parity-rs/crates/api-parity-rs/src/lib.rs` (+`dump_to_writer`) |
| `crates/api-parity-macros/src/lib.rs` | `api-parity-rs/crates/api-parity-rs-macros/src/lib.rs` |

Key rename across all four: `reference =` → `path =` on the
macro / decorator arg side.

---
> Source: [franciscoabsampaio/api-parity](https://github.com/franciscoabsampaio/api-parity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
