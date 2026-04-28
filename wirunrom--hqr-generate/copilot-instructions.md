## hqr-generate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`@wirunrom/hqr-generate` is a QR code generator and decoder with a Rust core compiled to WebAssembly, plus thin JS/React wrappers. Core philosophy: **binary-first** — core APIs return raw `Uint8Array` (PNG) or `string` (SVG); Base64/Data URL conversion is pushed to the UI layer.

## Build & Dev Commands

Building requires `wasm-pack` (and a Rust toolchain with `wasm32-unknown-unknown`).

- `npm run build` — full build: web WASM + Node WASM + React TS
- `npm run build:web` — `wasm-pack build --target web --out-dir pkg/web -- --features wasm,decode`
- `npm run build:node` — `wasm-pack build --target nodejs --out-dir pkg/nodejs -- --features wasm,decode`
- `npm run build:react` — `tsc -p tsconfig.react.json` (compiles `react-src/` → `react/`)
- `npm run clean` — removes `pkg/`
- `npm run prepare` — clean + build + strips `README.md`/`LICENSE`/`package.json`/`.gitignore` from `pkg/**`

Rust-only workflow (no wasm toolchain needed for these):
- `cargo check --features decode` — quick type check including decoder
- `cargo bench --bench generate` — criterion perf benchmarks (see `benches/generate.rs`)

There is no test runner configured. `tests/test.html` and `tests/my-app/` are manual smoke tests — serve over HTTP (WASM won't load from `file://`).

Publishing: `publish:npm`, `publish:github` (uses `.npmrc.github`), `release:test` (prerelease `next` tag).

## Architecture

### Rust core (`src/`)

The pipeline is split into two layers to keep fast paths cheap:

1. **Module grid** (cheap, ~`n²` bools where `n` ≈ 21–177):
   - `core::generate::generate_qr_modules(text, size, margin, ecc)` → `QrModules { n, margin, scale, dark: Vec<bool> }`
   - QR encoding uses the `fast_qr` crate (≈10× faster than `qrcode` for typical payloads). `QRBuilder::new(bytes).ecl(ecl).build()` returns a `QRCode` indexed as `qr[y][x] -> &[Module]`; `Module::value()` is the dark/light bool.
2. **Rasterization** (only when bytes are actually needed):
   - `render::png::render_png_modules(&QrModules)` — rasterizes **directly into a 1-bit packed PNG scanline buffer** (`BitDepth::One`). Uses `clear_range_1bit` to zero out pixel ranges byte-by-byte with head/middle/tail masking. No intermediate 8-bit bitmap is allocated on the fast path.
   - `render::svg::render_svg_modules(&QrModules)` — emits a single `<path>` with one rectangular subpath per horizontal run of dark modules. Uses relative `m` commands and tracks pen position to keep the `d` attribute compact.
   - `core::generate::rasterize(&QrModules)` → `QrBitmap` and `render::png::render_png(&QrBitmap)` / `render::svg::render_svg(&QrBitmap)` are the legacy 8-bit paths, kept for API compatibility but not used by the wasm entry points.

Other files:
- `src/core/types.rs` — `QrBitmap`, `QrModules` (with `img_size()` / `is_dark(x,y)` helpers), `DecodeInput` (`ImageBytes(&[u8])` or `Rgba { width, height, data }`).
- `src/core/decode/` — behind `decode` feature. `impl_.rs` computes BT.601 luminance inline inside the `rqrr::PreparedImage::prepare_from_greyscale` closure (integer shifts, no intermediate grayscale `Vec`).
- `src/wasm.rs` — `#[wasm_bindgen]` entry points (`generate`, `generate_png`, `generate_svg`, `decode`). `generate`/`generate_png` call `generate_qr_modules → render_png_modules` (1-bit path); `generate_svg` calls `generate_qr_modules → render_svg_modules`. `decode` dispatches on `input.is_instance_of::<Uint8Array>()` vs. reading `width`/`height`/`data` via `Reflect` (handles `ImageData`).
- `ecc` is passed over the FFI boundary as a `u8`: `0=L, 1=M, 2=Q (default), 3=H` (see `ecc_to_ecl` in `generate.rs`). The JS shims map the `'L'|'M'|'Q'|'H'` string to this `u8` — **do not pass the string directly to WASM**.

### Cargo features
- `generate` (default) — encode only, no decoder deps.
- `decode` — pulls in `image` + `rqrr`.
- `wasm` — pulls in `wasm-bindgen` + `js-sys` and exposes `src/wasm.rs`.

`fast_qr` is imported with `default-features = false` to avoid pulling in its own SVG/image builders (we render ourselves). The published build enables `wasm,decode`. `generate` is default so `cargo check`/`cargo build` work without the WASM toolchain.

`[profile.bench]` enables `lto = true` + `codegen-units = 1` for realistic perf numbers.

### JS entry points

Two hand-written shim files wrap the generated WASM bindings:
- `index.web.js` — browser ESM entry. Imports `pkg/web/hqr_generate.js`, lazily calls `init()` once (cached in `_initPromise`), then delegates. All functions are async.
- `index.node.js` — Node entry. Imports `pkg/nodejs/hqr_generate.js` (no init needed); functions are sync but typed as possibly-Promise.
- Both shims share a `normalizeOpts(opts)` helper that:
  1. Applies defaults: `size=320, margin=4, ecc='Q'`. **Calling `generate("x")` with no opts must not throw.**
  2. Translates `ecc` via `ECC_MAP = { L:0, M:1, Q:2, H:3 }` before passing to WASM. The WASM binding takes a `u8`; passing the raw string `'Q'` silently falls through to the default Q branch of the Rust match (this was a real bug — don't reintroduce it).
- `package.json` `exports` map selects: `browser`/`import` → web, `require`/`node` → node.

### React layer (`react-src/` → `react/`)

Compiled separately via `tsconfig.react.json`. `react` is an optional peer dependency.

- `useGenerate` / `useGenerateSvg` — **destructure `opts.size/margin/ecc` into primitive effect deps**. Passing the whole `opts` object caused WASM to re-run every render when callers inlined opts (e.g. `useGenerate(t, { size: 320 })`). If you add a new option, add it to the destructure *and* the dep array.
- Hooks pass `Uint8Array` directly to `new Blob([result as BlobPart], { type: 'image/png' })` — no `ArrayBuffer` slice copy. The `BlobPart` cast is for TS's `SharedArrayBuffer` variance, runtime is unaffected.
- Object URL is revoked in the effect cleanup.
- `useDecode` takes `ImageData` directly; no deep-compare on `input` (expensive + callers typically memoize).

## Conventions

- Keep the core binary-first. Do not add Base64/Data URL helpers to the Rust core or JS shims — that belongs in the React layer or caller code.
- Output is intentionally **black & white only** (scan reliability). Don't add color options without a strong reason.
- Prefer the module-level fast paths (`generate_qr_modules` + `render_*_modules`) for any new rendering. The legacy `QrBitmap` path is kept for back-compat only.
- When touching the WASM API surface, changes must propagate through: `src/wasm.rs` → `index.web.js` + `index.node.js` (incl. `normalizeOpts`) → `index.d.ts` → `react-src/` hooks (incl. dep arrays).
- `pkg/` is generated — never edit by hand; it's cleaned by `npm run clean`.

---
> Source: [wirunrom/hqr-generate](https://github.com/wirunrom/hqr-generate) — distributed by [TomeVault](https://tomevault.io/claim/wirunrom).
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
