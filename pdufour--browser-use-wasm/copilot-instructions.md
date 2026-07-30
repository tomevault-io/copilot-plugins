## capture-no-demo-hacks

> Capture pipeline must stay generic — no fixture or ShopDemo tuning


# Capture: no demo hacks

SnapDOM capture must match **any** website in `#capture-target`. Fixes belong in the **generic capture pipeline only** (`src/snapdom/capture.ts`, `src/snapdom/encode-worker.ts`, `examples/operator/ui/main.js` capture path, display sizing) — not the page under capture.

## Forbidden

- Edits to `examples/operator/fixtures/shop-demo/` or captured site HTML/CSS to fix alignment
- Demo-specific tuning: ShopDemo, checkout layout, button names, or matrix winners valid only on the built-in sample
- `CAPTURE_HEIGHT_FUDGE_PX`, `+1` height fudge, `zoom: 1.001`, mirror canvas overlays to fake live/snapshot match

## Allowed (generic pipeline)

- `#capture-target` measurement and clip
- SnapDOM params
- Viewport lock (integer CSS px on the measured box)
- 1:1 canvas display on the screenshot panel

Must work for any website — not just the built-in sample page.

## Visual-diff overfitting

Grid winners on the built-in fixture must not become global capture hacks (fixture-only vs generic-candidate — the experiment tooling is retired, archived notes in `docs/archive/`).

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
