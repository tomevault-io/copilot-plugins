## frust

> Frust is a Rust-native, mobile-first declarative UI framework: a `View`/`Widget` retained tree with

# Frust

Frust is a Rust-native, mobile-first declarative UI framework: a `View`/`Widget` retained tree with
signal-based reactivity, rendered via the frust-owned `frust-engine` strip pipeline on wgpu, with
Android/iOS/desktop shells, a plugin tier, and CLI/TUI tooling. It is a multi-crate Cargo workspace
documented hub-and-spoke — this file is the index; follow a link below rather than reading source
cold.

## Documentation

| Topic | Doc |
|-------|-----|
| Architecture (root index, cross-unit shape) | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) |
| CORE — view/widget lifecycle, layout, reactive, facade | [docs/CORE_ARCHITECTURE.md](docs/CORE_ARCHITECTURE.md) |
| RENDER — GPU render pipeline, text shaping | [docs/RENDER_ARCHITECTURE.md](docs/RENDER_ARCHITECTURE.md) |
| WIDGETS — baseline widget set, theme (Material/Cupertino/Glyph/shadcn/beUI ship as sibling plugin crates) | [docs/WIDGETS_ARCHITECTURE.md](docs/WIDGETS_ARCHITECTURE.md) |
| SHELLS — desktop/Android/iOS host integration | [docs/SHELLS_ARCHITECTURE.md](docs/SHELLS_ARCHITECTURE.md) |
| PLUGINS — OS-capability plugins | [docs/PLUGINS_ARCHITECTURE.md](docs/PLUGINS_ARCHITECTURE.md) |
| NATIVE_WIDGETS — native-control plugin | [docs/NATIVE_WIDGETS_ARCHITECTURE.md](docs/NATIVE_WIDGETS_ARCHITECTURE.md) |
| CLI — `frust` command + drive library | [docs/CLI_ARCHITECTURE.md](docs/CLI_ARCHITECTURE.md) |
| TUI — terminal workbench | [docs/TUI_ARCHITECTURE.md](docs/TUI_ARCHITECTURE.md) |
| DEVTOOLS — wire protocol + in-app debug service | [docs/DEVTOOLS_ARCHITECTURE.md](docs/DEVTOOLS_ARCHITECTURE.md) |
| Coding conventions (shared across units) | [docs/CODE_STANDARDS.md](docs/CODE_STANDARDS.md) |
| Build, run, test, environment | [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) |
| Test tiers and gates | [docs/TESTING.md](docs/TESTING.md) |
| Accepted limitations register | [docs/LIMITATIONS.md](docs/LIMITATIONS.md) |
| Contributing (branching, PR rules) | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Release procedure | [docs/RELEASING.md](docs/RELEASING.md) |
| Review priorities and hot spots | [docs/REVIEW_FOCUS.md](docs/REVIEW_FOCUS.md) |
| Doc structure/budget record | [docs/DOC_POLICY.md](docs/DOC_POLICY.md) |

Per-unit `*_DEVELOPMENT.md` / `*_CODE_STANDARDS.md` spokes hang off those last two indexes.

## Must-Know Commands

```
cargo test --workspace && cargo clippy --workspace --all-targets -- -D warnings && cargo fmt --check
```

This is the standard verify gate. [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) § Test is canonical —
it also covers the standalone-workspace gates (e.g. `huddle`/`clean-signals-frust`).

## Agent Guardrails

- Version pins are LAW (see [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) § Version-Pin Policy) — never
  bump one independently. `wgpu` is frust-owned (no `vello` constraint above it); still bump it only
  with the engine gate suite (RENDER_DEVELOPMENT.md), never casually.
- `examples/huddle`, `examples/shadertoy`, `examples/glyph-catalog`, `examples/playground`,
  `examples/design-system-sample`, `examples/material3-demo`, `examples/native-widgets-demo`, and
  `plugins/clean-signals-frust` are standalone workspaces excluded from the root graph — run their
  gates from their own directories.
- `workflow/` is a separate nested repo — never commit it.
- Doc edits must respect the budgets recorded in [docs/DOC_POLICY.md](docs/DOC_POLICY.md).
- `clean-signals` has one shared version requirement across three sites — see
  [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) § Version-Pin Policy before changing any of them.
- `frust-engine` (on `frust-gpu`) is the only renderer `frust-render` contains — not a cargo
  feature, not an override. Do not reintroduce a render-tier choice (env var, CLI flag, or
  feature) without an explicit new plan.
- Pull requests target `main`, and commits carry no assistant attribution or agent identity
  (`scripts/ci/commit-hygiene.sh` enforces it).

---
> Source: [frust-rs/frust](https://github.com/frust-rs/frust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
