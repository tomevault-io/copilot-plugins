## solitaire-king

> Project notes for working in this repo.

# CLAUDE.md

Project notes for working in this repo.

## GUI web build (macroquad + WASM)

The `klondike-gui` binary targets both native and WebAssembly (WebGL2). The web
build is deployed to GitHub Pages by `.github/workflows/deploy-pages.yml`.

`web/mq_js_bundle.js` is the miniquad JS glue, **vendored** (copied into the repo)
so the deployed page has no external runtime dependencies. This file is
version-matched to the miniquad version that macroquad pulls in.

**When bumping the `macroquad` version in `Cargo.toml`, re-vendor the JS glue:**

```sh
curl -sL -o web/mq_js_bundle.js https://not-fl3.github.io/miniquad-samples/mq_js_bundle.js
```

If the glue and the wasm disagree on the ABI (`version=N` near the top of the
file), the game will fail to start or misbehave on web only (native is
unaffected). It's currently `version=2`, matching macroquad 0.4.x.

## Web assets path

The GUI calls `set_pc_assets_folder("assets")`, which prefixes every asset load
with `assets/` **on the web target too** (despite the "pc" in the name). The
deploy workflow therefore serves assets under `dist/assets/`, not the site root.
Keep those in sync if either the folder call or the workflow's copy step changes.

---
> Source: [sandersonb/solitaire-king](https://github.com/sandersonb/solitaire-king) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
