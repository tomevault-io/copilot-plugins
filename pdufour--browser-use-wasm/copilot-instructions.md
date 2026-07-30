## client-side-only

> Everything runs in the browser — no server-side inference or app logic


# Client-side only (non-negotiable)

**The product runs entirely in the user’s browser.** Capture, model load, VLA grounding, coordinate parsing, and the click overlay all execute **client-side** — main thread or **Web Workers** (`wllama/worker.js`, `snapdom/encode-worker.js`). There is no backend that runs the app for the user.

## Runs in the browser (required)

| Concern | Where |
|--------|--------|
| Page capture | Main thread — SnapDOM (`examples/operator/ui/main.js`) |
| VLA load + inference | **Browser WASM** — `src/wllama/worker.js` + `@wllama/wllama` |
| PNG encode | `src/snapdom/encode-worker.ts` |
| Grounding prompts / parse | Imported into worker; **never** Node |
| GGUF weights | **Pre-cached** in `.model-cache/` via `npm run cache:model` (Node); browser prefers `/model-cache/`, or downloads public registry HF URLs on demand (still client-side) |
| `wllama.wasm` | Same-origin `/wllama/wllama.wasm` (Vite serves from `node_modules`) |

Serve the app with **`npm run dev`** or **`npm run preview`** so COOP/COEP headers apply. Open in **Chrome or Edge** (WebGPU + JSPI). **Not** `file://`.

## Node.js is dev tooling only — not the product runtime

Node may appear in the repo **only** for:

- **Vite** — static dev server + headers (`vite.config.js`)
- **`npm run cache:model`** — download GGUFs into `.model-cache/` (no inference)
- **Playwright** — drive real Chrome to validate the **client** app (`npm run test`)

**Never** run wllama, llama.cpp, or VLA grounding inside Node (`node -e`, Express API, child_process to CLI as the app path). **Never** suggest “run inference on the server and return coords to the client.”

## Forbidden

- Python / PyTorch / FastAPI / Gradio / remote inference APIs for grounding
- Node servers that load GGUF or call wllama for user requests
- SSR or edge functions that run the model
- DOM-derived grounding coords — see `no-dom-grounding.mdc`
- Registry models — see `vla-registry.mdc`

## If something “only works in Node”

That is a bug. Fix browser WASM/worker config (`wllama/browser-shim.js`, `ENVIRONMENT_IS_NODE=false` patch, load params in `config/vl.js` / per-model registry) — do not move inference off the client.

## Related rules

- `wllama-only.mdc` — VLA models via wllama in a worker
- `no-dom-grounding.mdc` — coords from screenshot inference only
- `blackbox-e2e.mdc` — prove it in real Chrome, not Node unit tests

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
