## wllama-only

> VLA models via wllama in the browser only — never Node/Python inference


# wllama in the browser only

**All inference is client-side.** See `.cursor/rules/client-side-only.mdc`.

Registry **GUI/VL models** (default ShowUI-2B) use `@wllama/wllama` (llama.cpp **WASM** inside a **dedicated Web Worker**). The main thread only does UI + SnapDOM; it never calls `createChatCompletion`.

## Required (browser worker)

- Inference **only** in `src/wllama/worker.js` — import `./wllama-browser-shim.js` first
- RPC from main thread via `src/wllama/client.js` (`Worker`, not Node `worker_threads`)
- WASM URL: `/wllama/wllama.wasm` (same-origin; Vite `vite.wllama-wasm.js`)
- Vite forces `ENVIRONMENT_IS_NODE=false` in `@wllama/wllama` so llama.cpp never uses Node pthreads
- Load: per-model `n_ctx`, `image_min_tokens` / `image_max_tokens`, `n_gpu_layers` from `src/config/models/*.js` (shared defaults in `config/vl.js`), `n_threads: 1`
- Default ShowUI-2B navigation layout (card-verbatim `_NAV_SYSTEM` → task → image) in `src/actions/navigation.ts`; worker stays generic (`completion()` only)
- Weights: **pre-cached** in `.model-cache/` (`npm run cache:model`); browser loads `/model-cache/` only — Node downloads, not runtime HF fetch on Load

## Forbidden

- **Node wllama / llama.cpp** for user-facing grounding (no `require('@wllama/wllama')` in `scripts/` for inference)
- Python inference sidecars — see `client-side-only.mdc`
- Server-side or Python VL inference — see `vla-registry.mdc` and `client-side-only.mdc`
- DOM grounding cheats — see `no-dom-grounding.mdc`
- Main-thread `new Wllama()` for production inference

## Validation

`npm run test` (one E2E in real Chrome) — see `blackbox-e2e.mdc`. CLI `llama-mtmd-cli` is debug-only, not a product path.

## If grounding fails

Fix browser worker/WASM/template/vision tokens — **not** Node or Python offload.

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
