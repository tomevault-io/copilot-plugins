## edgeflow-js

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- **Build:** `npm run build` (runs `tsc` then `scripts/build-browser.js` which produces `dist/edgeflow.browser.js` via esbuild; `onnxruntime-web` is marked external).
- **Watch compile:** `npm run dev`
- **Lint:** `npm run lint` (ESLint on `src/**/*.ts`)
- **Unit/integration tests (vitest, happy-dom):** `npm test` / `npm run test:unit` / `npm run test:integration`
  - Single test file: `npx vitest run tests/unit/tokenizer.test.ts`
  - Single test by name: `npx vitest run -t "test name pattern"`
- **E2E (Playwright, Chromium):** `npm run test:e2e`
  - Uses `playwright.config.ts` by default; alternate configs exist for `localai`, `network`, `privatedoc` scenarios (run with `npx playwright test -c playwright.localai.config.ts`).
  - Playwright auto-starts `npm run demo:server` on `localhost:3000`.
- **Demo app:** `npm run demo` (builds then serves `demo/server.js` on port 3000). Load a Hugging Face ONNX URL in the browser UI to exercise pipelines.
- **Docs (VitePress):** `npm run docs:dev` / `npm run docs:build`

## Architecture

edgeFlow.js is a browser-first ML inference framework. The runtime graph is: **Pipeline → BasePipeline → RuntimeManager → Runtime backend (ONNX/WebGPU/WebNN/WASM) → Scheduler → MemoryManager**. All public exports flow through `src/index.ts`.

### Layered structure (`src/`)

- **`core/`** — framework internals. `types.ts` is the canonical type/error surface (`EdgeFlowError`, `ErrorCodes`, all `Tensor`/`Runtime`/`Pipeline` interfaces — most other files import from here). `runtime.ts` holds the `RuntimeManager` singleton, runtime factory registry, and priority-based automatic backend selection (webgpu > webnn > wasm). `scheduler.ts` implements the global priority queue / concurrency-limited `InferenceScheduler` that every runtime dispatches through. `memory.ts` provides `MemoryManager`, `MemoryScope`, and `ModelCache` with reference-counted cleanup. `composer.ts` enables `compose()`/`parallel()` multi-stage pipelines. `plugin.ts` is the extension point for third-party pipelines/backends/middleware. `device-profiler.ts` recommends quantization/model variant based on device tier. `worker.ts` runs inference in a Web Worker.

- **`backends/`** — concrete `Runtime` implementations: `onnx.ts` (onnxruntime-web, peer dep), `webgpu.ts`, `webnn.ts`, `wasm.ts`, plus `transformers-adapter.ts` for interop with transformers.js. `registerAllBackends()` wires factories into `RuntimeManager`.

- **`pipelines/`** — task-specific wrappers extending `base.ts`'s `BasePipeline`. The `pipeline(task, options?)` factory in `index.ts` looks up a registered pipeline factory (built-in or plugin) and returns a ready-to-run instance. Each pipeline owns its own tokenizer/preprocessor, model loading, and result formatting.

- **`utils/`** — `tokenizer.ts` (BPE, WordPiece, Unigram — loads `tokenizer.json` directly), `preprocessor.ts` (image/audio/text), `model-loader.ts` (preloading, sharding, resumable downloads), `cache.ts` (`InferenceCache`, `ModelDownloadCache` — IndexedDB-backed), `hub.ts` (HuggingFace Hub download helpers + `POPULAR_MODELS`), `offline.ts`.

- **`tools/`** — developer tooling surface: `quantization.ts` (int8/uint8/float16 quant + dequant), `debugger.ts` (tensor inspection, histograms, heatmaps, trace events), `monitor.ts` (`PerformanceMonitor` + dashboard generators), `benchmark.ts`.

### Cross-cutting conventions

- **ESM only** (`"type": "module"`, `sideEffects: false`). All intra-repo imports use `.js` extensions even from `.ts` source — required for Node ESM resolution after `tsc` emits.
- **`onnxruntime-web` is an optional peer dep** marked `external` in the browser bundle; consumer bundlers resolve it. Do not import it eagerly in code paths that should run without ONNX.
- **Errors always use `EdgeFlowError` + `ErrorCodes`** from `core/types.ts` — do not throw bare `Error` from library code.
- **Scheduling is mandatory:** runtime inference paths go through `getScheduler()`. New backends should dispatch via the scheduler rather than calling model.run directly, so priority/concurrency controls are honored.
- **Tests:** unit/integration run under happy-dom (no real WebGPU/WebNN); those backends are exercised via Playwright E2E against the demo server. Test timeout is 30s.

---
> Source: [s-zx/edgeFlow.js](https://github.com/s-zx/edgeFlow.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
