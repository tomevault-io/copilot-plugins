## vla-registry

> Multi-model GUI/VL registry — browser wllama grounding (ShowUI-2B is default + E2E gate)


# VLA model registry (browser)

**Runs client-side in the browser** — see `.cursor/rules/client-side-only.mdc`.

This app is a **VLA click finder**: SnapDOM capture → **GGUF vision-language models** in `src/config/models/registry.js` via wllama (WASM worker). Users pick a model in the UI; weights are pre-cached in `.model-cache/`.

## Default and CI

- **Default model:** `ShowUI-2B` (`src/config/models/ShowUI-2B.js` card + `src/config/vl.js` shared defaults).
- **E2E gate:** `ShowUI-2B` only (green circle) — see `.cursor/rules/blackbox-e2e.mdc`. Other models are manual/experimental until tuned.

## Registry models

- Add/edit entries in `src/config/models/*.js` and `src/config/models/registry.js`.
- Cache: `npm run cache:model -- --model <id>`, `npm run cache:public`, or `cache:all` with `HF_TOKEN`.
- Browser prefers **same-origin `/model-cache/`**; public models may **download from registry HF URLs on demand** (`src/wllama/model-sources.ts`). Gated models and `?e2e=1` / `?cacheOnly=1` stay cache-only.

## UI and naming

- Use each model’s **registry `label`** in status, errors, and logs — not a generic “ShowUI only” message.
- For ShowUI-branded weights, prefer **ShowUI** in user-facing copy (not “Qwen2-VL chat”).
- Mark non–E2E-validated models **(experimental)** in the switcher.

## Forbidden

- Python / server-side VL inference for grounding
- DOM-derived coordinates — see `no-dom-grounding.mdc`
- Removing the model registry or pretending the app is single-model only

## Validation

`npm run test` — ShowUI-2B ×3 E2E in real Chrome.

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
