## qr-code-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Context
A 100% client-side QR code generator: type text/URL, customize colors, toggle a transparent
background, drop a center logo, and export PNG (up to 4096px) or vector SVG. Nothing leaves
the browser. The UI is in Brazilian Portuguese (pt-BR); code and docs are in English.

## Stack
- Runtime: Node (dev), browser (prod). React 19 + TypeScript ~6.0
- Build: Vite 8 (`@vitejs/plugin-react`)
- Styling: Tailwind CSS v4 via `@tailwindcss/vite` — **no** `tailwind.config.js`, **no** PostCSS;
  utilities come from `@import "tailwindcss"` in `src/index.css`, tokens in the `@theme {}` block
- QR: the `qrcode` library
- Serving: static SPA served by nginx in a multi-stage Docker image. No backend, no `.env`.

## Dev
```bash
npm install        # once, after clone
npm run dev        # vite dev server (HMR) — does NOT type-check
npm run build      # tsc -b && vite build — a type error BREAKS the build
npm run lint       # eslint .
npm run preview    # serve the built dist/
```
There is no test suite configured.

```bash
SSH_HOST=myserver ./deploy.sh   # build + run on a remote host (rsync → docker build → docker run)
```

## Architecture
```
src/App.tsx     single component (~730 LOC) — the entire app
src/main.tsx    mounts <App/> in StrictMode
src/index.css   @import tailwind + @theme + .checkerboard (transparent preview)
index.html      loads the Inter font via Google Fonts
Dockerfile      multi-stage: node:22-alpine builds → nginx:alpine serves
nginx.conf      gzip + immutable cache for /assets/ + SPA try_files fallback
deploy.sh       rsync + docker build + docker run on a remote host (configured via env vars)
```

Core flow — everything is state-driven over a canvas:
- `drawQRWithLogo(canvas, size)` is the master render function (memoized via `useCallback`
  over all config state). It draws the QR onto the canvas, then post-processes: transparency
  and logo overlay.
- **Two canvases**: a 320px preview (re-rendered in a `useEffect` whenever `drawQRWithLogo`
  changes) and a detached canvas created on demand at export resolution (512/1024/2048/4096)
  inside `downloadPNG`.
- **SVG** (`downloadSVG`) does NOT use canvas: it re-generates the QR as a string via
  `QRCode.toString({type:'svg'})` and manipulates the SVG DOM (DOMParser) — removing
  background rects/paths for transparency and injecting `defs`/`clipPath` + a base64
  `<image>` for the logo.
- **History**: configs saved to `localStorage` under the key `qr-code-history`. Each item
  stores the full config + a 128px PNG thumbnail (data URL). Restoring re-applies the state;
  it does not re-generate the original output.

## Gotchas
- **Duplicated render logic**: the canvas (`drawQRWithLogo`) and SVG (`downloadSVG`) have
  *separate* implementations of transparency and the logo overlay. Any visual behavior change
  must be mirrored in both, or PNG and SVG will diverge.
- **Heuristic transparency**: on the canvas, pixels where R, G, B are all `> 240` become
  alpha 0. A light/near-white code color (`fgColor`) would be partially erased.
- **Dead canvas**: the `canvasRef` (hidden canvas in JSX, sized to `QR_SIZE`) is not used for
  export — `downloadPNG` creates a fresh canvas each click. `canvasRef` is effectively dead code.
- UI and dates are in pt-BR (`toLocaleDateString('pt-BR', ...)`); `lang="pt-BR"` in the HTML.
- Bleeding-edge versions (React 19, Vite 8, TS 6, ESLint 10) — check compatibility before bumping.
- Default error correction `'H'` — recommended when a logo covers the center.
- **`npm run dev` does not type-check; `npm run build` does** (`tsc -b` before vite). Code that
  runs in dev can still break the build/deploy.
- **Non-secure context (HTTP)**: when served over `http://` (e.g. on a LAN),
  `window.isSecureContext === false`, so secure-context-only APIs (`crypto.randomUUID`,
  `crypto.subtle`, `navigator.clipboard`) are `undefined` and throw. They work on `localhost`
  in dev but break in production. History IDs use `genId()` (fallback via
  `crypto.getRandomValues`) for this reason. Do not introduce those APIs without a fallback.

---
> Source: [viktorkav/qr-code-generator](https://github.com/viktorkav/qr-code-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
