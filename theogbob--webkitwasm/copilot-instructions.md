## webkitwasm

> A fully modern browser engine running INSIDE a browser tab. No VM, no x86

# BrowserInBrowser — WebKit compiled to WebAssembly

## Goal
A fully modern browser engine running INSIDE a browser tab. No VM, no x86
emulation: WebKit (WebCore + JavaScriptCore) is compiled to WASM with
Emscripten. The page hosts an interactable `<canvas>` that is the engine's
renderer. All networking leaves the tab over the **Wisp protocol**
(wisp-js server side; wisp-server-python rejected — Wisp v1-only).

## Architecture decisions (see docs/summaries/decision-001-architecture.md)
- **Engine**: WebKit (WebCore + JSC). Chosen for portability — it is the only
  major engine with a realistic direct-WASM story (JSCOnly/embedded port
  lineage, curl network backend, generic RunLoop).
- **JS execution**: JSC **CLoop interpreter** (JIT-less). WASM cannot self-JIT
  conventionally; CLoop is WebKit's supported no-JIT mode.
- **Process model**: single process, WebKit1-style thin embedder. We do NOT
  port the WebKit2 multiprocess/IPC layer. NOTE: no supported single-process
  embedding API exists outside Cocoa — we embed WebCore::Page + client
  interfaces against internal headers, modeled on the PlayStation port.
- **Port strategy**: Phase 1 builds JSCOnly; then a custom `PORT=Emscripten`
  (3-file pattern: `ALL_PORTS` + `OptionsEmscripten.cmake` +
  `PlatformEmscripten.cmake`), borrowing the Win port's curl glue.
- **Rendering**: Skia CPU raster (vendored `Source/ThirdParty/skia`) → blit
  to canvas. cairo is dead in WebKit ≥2.46 — do not invest in it. GPU path
  (Skia → WebGL2, CanvasKit-style) is a later phase.
- **Networking**: WebKit curl backend → curl 8.17 + OpenSSL 3.5 + nghttp2 →
  Emscripten SOCKFS with wisp-js `WispWebSocket` swapped in (libcurl.js
  pattern — pure JS shim, no C-level Wisp code). TLS terminates in-engine.
  OpenSSL not mbedTLS/BoringSSL — WebCore's curl backend hardwires the
  OpenSSL API (decision-003). Fontconfig is built for wasm, not patched out
  (WebCore's FontCacheSkia calls fontconfig directly).
- **Threads**: Emscripten pthreads, `-sPROXY_TO_PTHREAD`, pre-sized pool →
  SharedArrayBuffer → host page MUST be served with COOP/COEP headers.
  No Asyncify; JSPI only as a later optimization.

## Repo layout
- `src/` — our code: embedder shell, platform glue, Wisp bridge, host page.
- `third_party/` — WebKit checkout, emsdk, dep sources. **Git-ignored**;
  pinned by exact hash in `docs/summaries/00-project-brief.md`.
- `docs/summaries/` — project brief, decisions, handoffs (one active handoff).
- `docs/research/` — research reports (prior art, build surface, Wisp).
- `docs/archive/` — superseded docs; read only when told.

## Hard constraints
- Local git only. Never publish without explicit permission.
- No JIT in the guest engine — JS-heavy sites will be slow; that's accepted.
- wasm32 4 GB memory ceiling — Memory64 REJECTED (10–100% perf penalty, no
  Safari). Stay wasm32; trim features instead.
- Dev server must send COOP `same-origin` + COEP `require-corp`.

## Workflow
- Phase gates are fail-fast: Phases 0–2 exist to kill the approach cheaply if
  a load-bearing assumption breaks (see brief §Phases).
- Before modifying WebKit sources: record every patch as a `.patch` in
  `src/patches/` so the pinned checkout stays reproducible.
- Codex review before presenting any non-trivial code change.

---
> Source: [theogbob/WebkitWasm](https://github.com/theogbob/WebkitWasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
