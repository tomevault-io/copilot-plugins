## tinyimg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Astro dev server (localhost:4321)
npm run build     # Production static build → dist/
npm run preview   # Preview production build locally
```

No test framework is configured. Verify changes with `npm run build`.

## Architecture

Client-side image compression tool. Astro SSG for static HTML + SEO, React islands for interactivity, Web Worker for off-main-thread processing. Zero server — all compression happens in the browser.

### Island Architecture

- **Static Astro components** (`.astro`): Header, Hero, HowItWorks, FAQ, Footer, Layout — rendered at build time, zero JS shipped
- **React island** (`.tsx`): `ImageProcessor` with `client:visible` — only hydrated when scrolled into view. Contains all interactive UI: DropZone, ResultCard, BulkActions

### Processing Pipeline

`ImageProcessor.tsx` is the orchestrator. It spawns a **short-lived Web Worker** per batch:

1. User drops files → ImageProcessor validates (type, size ≤20MB, count ≤50)
2. Creates Worker from `src/workers/image-worker.ts`
3. Sends files sequentially via `postMessage`
4. Worker uses `OffscreenCanvas` + `convertToBlob()` for resize/compress/convert
5. **Smart compression**: tries multiple quality levels × multiple formats (original + WebP + JPEG for PNG), picks smallest result
6. Worker posts back blob + metadata, ImageProcessor updates React state
7. Worker terminated after batch completes

### Two-State UI

- **Before upload**: Full DropZone (corner bracket hover effect) + format pills
- **After upload**: DropZone disappears, centered inline table (`max-w-2xl`) with stats bar, file rows, compact "Add more" strip

### Key Files

- `src/components/ImageProcessor.tsx` — State hub: results[], format, processing, toasts, worker lifecycle
- `src/workers/image-worker.ts` — Smart compression: `smartCompress()` tries all codec/quality combos
- `src/lib/format-support.ts` — Format detection, AVIF support probe, constants (MAX_FILE_SIZE, MAX_FILE_COUNT, WORKER_TIMEOUT_MS)
- `src/lib/download.ts` — Single file download + JSZip bulk download
- `src/styles/global.css` — Tailwind v4 `@theme` block (dark/gold palette), CSS animations (shimmer, card-enter, progress-ring, etc.)
- `src/layouts/Layout.astro` — SEO: meta tags, OG, Twitter cards, WebApplication structured data, sitemap

## Tech Stack

Astro 5, React 18, Tailwind CSS 4 (via `@tailwindcss/vite`), TypeScript strict. No `tailwind.config.mjs` — theme defined in CSS `@theme` block. Deploy target: Cloudflare Pages (static).

## Style Conventions

- Dark theme: `#0a0a0a` bg, `#c9a84c` gold accent, `#141414` card surfaces
- Tailwind v4 custom colors: `bg-bg-primary`, `text-gold`, `border-border`, `text-success`, `text-error`
- React components use `jsx: "react-jsx"` — no `import React` needed
- ESM throughout (`"type": "module"` in package.json, `esModuleInterop: true`)

---
> Source: [michael-duo/tinyimg](https://github.com/michael-duo/tinyimg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
