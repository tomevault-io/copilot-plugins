## no-dom-grounding

> Client-side grounding from screenshot only — never live DOM


# No DOM cheating for grounding

**Client-side only** — see `.cursor/rules/client-side-only.mdc`.

Click coordinates must come **only** from the **loaded VLA + wllama (browser worker)** on the **SnapDOM screenshot buffer**. Never derive, fix, or validate grounding using live page DOM or a server.

## Allowed

- **SnapDOM** capture of `#capture-target` (screenshot pixels).
- **VLA inference** in `wllama/worker.js` → `[x, y]` normalized 0–1 on that screenshot.
- **Overlay** on `#screenshot-img`: map model norm coords → pixels on the **displayed screenshot** (`getBoundingClientRect` of the screenshot image element only).
- **E2E**: UI clicks, `#raw-output` text, `#click-marker` vs screenshot image — **pixel samples on the screenshot at the model’s point** (canvas/`naturalWidth`).

## Forbidden (production and tests)

- Reading **live** `#btn-submit`, `#btn-cancel`, `getBoundingClientRect()` on page controls to **place or score** the marker.
- `querySelector` / element IDs to **compute** expected `[x, y]`.
- Mock/fake wllama clients that return DOM-derived coordinates.
- Importing `src/actions/navigation.ts` in Playwright specs to assert internal helpers (see `blackbox-e2e.mdc`).
- Label-text/selector control lookup to **execute** form actions — execution acts on the element at the grounded point only (see `vision-only-execution.mdc`).

Capture may use element **size** (`offsetWidth` / `offsetHeight`) only to choose SnapDOM resolution — not for grounding coords.

## E2E grounding checks (blackbox)

1. `#raw-output` contains `Parsed click` and a `[x, y]` in `[0, 1]`.
2. `#click-marker` visible on the screenshot panel.
3. **Screenshot pixels** at the marker (from model output): e.g. Submit → greenish (`G` dominant); Cancel vs Submit → markers **> 0.08** apart on same capture.

Do **not** compare marker position to live DOM layout regions (`groundingHitsTarget`, `getCaptureLayout`, `assertMarkerTargetsRegion`).

## If accuracy is poor

Fix wllama load params, vision resize, prompt/prefill, and capture quality — **not** DOM snapping or Python sidecars.

## If “nothing works” in the browser

1. `npm run dev` → open **http://127.0.0.1:5173/** (not `file://`, not Live Server on `dist/`).
2. **Chrome or Edge** only; enable GPU acceleration in system settings.
3. `npm run cache:model` once; wait for status **loaded in browser WASM worker**.
4. **Capture page** before **Run task**; goal should be a short instruction (`click Submit`).
5. If load says `abort signal` → hard-refresh, click **Load** again (app retries fresh cache once).

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
