## drizzle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Drizzle CSS — a classless-first CSS framework. Source CSS modules in `css/` are bundled and minified **once at release time** by the `drizzle-build` binary (`src/main.rs`, using the [Lightning CSS](https://lightningcss.dev/) crate) into `dist/drizzle.css` + `dist/drizzle.min.css`, which are **committed**. The `drizzle_css` library crate embeds those committed files as `&'static str` constants via `include_str!`. Consumers of the library pull a pure `&'static str` with no lightningcss transitive tree. Demo HTML pages in `demo/` link to the built artifacts in `dist/`.

## Commands

```sh
cargo run --release --features build --bin drizzle-build   # rebundle css/ → dist/ (regenerate after any css/ edit)
cargo build --release                                      # build the library (embeds committed dist/)

python3 -m http.server 8765      # preview demos at http://localhost:8765/demo/index.html
                                 # (file:// works too but Chrome may block it under Playwright)
```

No test suite, no linter. CSS validity is enforced at bundle time — Lightning CSS surfaces parse errors and fails the `drizzle-build` run. CI (`.github/workflows/ci.yml`) re-runs the bundler and fails on `git diff dist/`, so a `css/` edit committed without regenerating `dist/` is caught.

## Architecture

### Build pipeline

The bundling lives in the **`drizzle-build` bin (`src/main.rs`)**, which runs `lightningcss::bundler::Bundler` against `css/drizzle.css`, inlining every `@import`, then prints the stylesheet twice — once unminified, once minified — with browser targets pinned to Chrome 117 / Firefox 125 / Safari 17.5 / Edge 117 — the floor is set by the newest feature the CSS ships unpolyfilled. Lightning CSS down-levels *syntax* (e.g. nesting is flattened for these targets), but it does **not** polyfill *semantics*: `:has()`, `color-mix()`, `[popover]`, and `text-wrap: pretty` pass through untouched, so the targets must be new enough to actually support them or the baseline is a lie. That constraint (popover: Chrome 114 / Safari 17 / FF 125; `text-wrap: pretty`: Safari 17.5) is what pins the numbers. The bin writes `dist/drizzle.css` + `dist/drizzle.min.css`; `src/lib.rs` embeds them with `include_str!("../dist/...")` as the `CSS` / `CSS_MIN` constants.

lightningcss is an **optional** dependency gated behind the `build` feature, and the bin is `required-features = ["build"]` — so the default library build (what consumers get) never compiles lightningcss. Only the release-time bundling path (`--features build`) does. A change to `css/**` requires re-running the bin (`cargo run --release --features build --bin drizzle-build`) and committing the regenerated `dist/`.

### Source CSS layering — cascade layers decide who wins

`css/drizzle.css` is the single entry point. It declares an explicit `@layer` order, then `@import`s every module into its matching layer:

```
tokens → dark → reset → base → typography → lists → tables → forms → media → utilities → print → palette
```

**Cascade layers — not specificity, not source order — are the strategy.** Every selector is bare-element or single-class (e.g. `.card`, `.bg-red`), so specificity is mostly flat; the `@layer` order in `drizzle.css:7` is what resolves conflicts. A later layer beats an earlier one. Two rules matter most:

1. **`palette.css` MUST stay in the last layer.** It defines `.bg-red`, `.bg-blue`, etc. If anything that sets `background` (like `.card` in `utilities.css`) landed in a later layer, it would silently override the hue on `<div class="card bg-red">`. Keep palette last in both the `@layer` declaration and the import list.
2. Tokens come first because every later rule references `var(--...)` from them.

Note: because the import list and the `@layer` declaration are kept in the same order, source order and layer order agree — but layer order is the one that's load-bearing. `!important` inverts layer precedence (earlier layers win), which is why `print.css` uses `!important` to force print styles regardless of its layer position.

### Classless-first contract

Every visible semantic HTML element is styled by a **bare element selector**, never via a required class. Adding a class is opt-in enhancement. When editing `base.css`, `typography.css`, `lists.css`, `tables.css`, `forms.css`, or `media.css`, keep selectors classless. Reserve class selectors for `palette.css` (color modifiers) and `utilities.css` (layout/component helpers).

### Theming via CSS variables

All visual values live in `css/tokens.css` as custom properties on `:root` — palette hues, semantic aliases (`--color-error`, `--color-warning`, `--color-success`, `--color-info`, `--color-question`), surface/text/border colors, font families, type scale, weights, gutters, borders, radii, shadows, layout widths. Downstream consumers retheme by overriding a single var; no Sass, no build step on their side.

### Dark mode — two parallel paths

`css/dark.css` declares the same dark overrides in two trigger blocks:
- `@media (prefers-color-scheme: dark) { :root:not(.light) { ... } }` — auto-respect OS, but `<html class="light">` opts out.
- `:root.dark { ... }` — explicit class override regardless of OS.

Both blocks map active vars (`--color-bg`, `--shadow`, etc.) to defaults declared once in `css/tokens.css` as `--dark-*` tokens. End users retheme dark mode by overriding a single `--dark-*` var. When adding a new themed value, add the `--dark-*` default to `tokens.css` and the `--color-x: var(--dark-color-x)` mapping to **both** trigger blocks — values are now single-sourced, but var **names** must stay in sync between the two triggers.

### Bundler output is committed

`dist/drizzle.css` + `dist/drizzle.min.css` are **committed** (only `target/` is gitignored). They are the release artifacts the library embeds, so they must stay in sync with `css/`. After any `css/**` edit, regenerate and commit them (`cargo run --release --features build --bin drizzle-build`); CI fails the build if committed `dist/` drifts from a fresh bundle. This is deliberate: consumers get exactly the bytes tested at release with no lightningcss in their dependency graph — see [issue #1](https://github.com/thombruce/drizzle/issues/1).

---
> Source: [thombruce/drizzle](https://github.com/thombruce/drizzle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
