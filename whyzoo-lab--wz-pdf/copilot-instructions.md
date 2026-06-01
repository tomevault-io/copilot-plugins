## wz-pdf

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Compile Electron main (tsc) + start Vite dev server + launch Electron
npm run dev:vite     # Vite only (web browser, no Electron)
npm test             # Vitest in watch mode
npm run test:run     # Vitest single run (CI)
npx vitest run src/path/to/file.test.tsx   # Run a single test file
npm run build        # Production build (tsc + vite build)
npm run build:exe    # Build both Windows artifacts (portable first, then NSIS installer
                     # — the NSIS afterPack hook bundles the portable as viewer-template.exe)
npm run lint         # ESLint
```

`predev` automatically compiles `electron/` before `dev` runs — no need to call `electron:compile` manually in development.

## Architecture

### Dual-process Electron app

```
electron/main.ts      ← Node/Electron main process (BrowserWindow, IPC, embedded-PDF extraction)
electron/preload.ts   ← Context bridge (exposes IPC to renderer)
src/                  ← Renderer process (React app served by Vite on :5173)
```

The renderer never uses Node APIs directly. All IPC calls go through `window.electronAPI`:

| API | Description |
|---|---|
| `onOpenFile(cb)` | File path from OS open-file event, CLI arg, or `.pdf` file-association entry point |
| `onOpenPdfBytes(cb)` | PDF bytes when launched as a viewer-exe (portable build only) |
| `readFile(path)` | Read a local file via main process (avoids CORS on `http://localhost`) |
| `exportExe(pdfData)` | Save current PDF embedded into a copy of the portable exe |
| `printWindow()` | Native OS print dialog |

### Rendering pipeline

```
PDF bytes → pdfjs-dist (Worker) → HTMLCanvasElement → Konva Stage (KonvaImage)
```

Key points:
- **pdfjs worker** is loaded via a blob URL wrapper in `src/main.tsx` that polyfills `Uint8Array.prototype.toHex` and `Map.prototype.getOrInsertComputed` before importing the real worker — both methods are absent in the Electron Chromium version but required by pdfjs 5.x.
- `usePdfPage` renders each page once and stores the result in a **module-level WeakMap cache** (`pageCache`). View-mode switches (single ↔ spread ↔ grid ↔ fullscreen) do not re-render pages.
- `PdfPage` passes `pageData.canvas` directly to `<KonvaImage>` — no `toDataURL` / `new Image()` round-trip.

### Coordinate system

Annotations are stored in **PDF-point space** (independent of zoom/scale).

- `effectiveZoom = PDF_RENDER_SCALE * zoom` (render scale 1.5 × user zoom)
- Screen → stored: `toStoredCoords(x, y, effectiveZoom)` → divides by effectiveZoom
- Stored → screen: `toScreenCoords(x, y, effectiveZoom)` → multiplies by effectiveZoom
- For pdf-lib export, Y-axis is flipped: `toPdfLibY(pdfJsY, height, pageHeight)`

### State management (App.tsx)

All state lives in `App.tsx` (no external store). Key state:

| State | Purpose |
|---|---|
| `file / fileBytes` | Current PDF — `fileBytes` kept separately for export |
| `zoom` | Display zoom multiplier (0.1–3, step 0.25). Does NOT affect render cache. |
| `rotation` | Page rotation: `0 \| 90 \| 180 \| 270` degrees |
| `viewMode` | `'single' \| 'spread' \| 'grid' \| 'fullscreen'` |
| `fullscreenLayout` | `'single' \| 'spread'` — captured from `viewMode` when entering fullscreen |
| `appMode` | `'viewer' \| 'editor'` — hides annotation tools in viewer mode |
| `activeMode` | `'select' \| 'stamp' \| 'signature' \| 'watermark' \| 'pen' \| 'rectangle' \| null` |
| `pendingStamp / pendingSignature` | Image data URL awaiting placement click |
| `scrollToPage` | Target page number for programmatic scroll (cleared after use) |
| `currentPage` | Currently visible page number (1-based) |
| `isExporting` | Boolean blocking UI during export operations |
| `isPanelOpen` | Whether the left Pages panel is open |
| `isPageOperating` | Boolean blocking UI during page insert/delete/reorder operations |
| `toast` | `{ id: number; message: string } \| null` — current toast notification |

### Component tree

```
App
├── ActionBar          ← Top bar: view/zoom controls, editor tools, Reset markup button, upload/export
├── PagePanel          ← Left sidebar: thumbnail strip, multi-select, drag-reorder, add/delete pages
│                        readOnly={appMode === 'viewer'} — visible in both viewer and editor modes
├── Toast              ← Fixed bottom-center auto-dismissing notification (2500ms)
├── [hidden file input] ← Double-click empty area to open
└── main
    └── PdfViewer
        ├── (single)   → LazyPdfPage × numPages (IntersectionObserver-gated)
        ├── (spread)   → SpreadView → LazyPdfPage pairs
        ├── (grid)     → GridView → LazyPdfPage × numPages at zoom=0.3
        └── (fullscreen) → FullscreenView → PdfPage(s) (1 or 2 depending on fullscreenLayout)
```

`LazyPdfPage` wraps `PdfPage` and defers mounting the Konva Stage until the container enters the viewport (400px rootMargin). Once mounted it stays mounted — the render cache makes re-mounts cheap.

`PdfPage` = Konva `<Stage>` with three layers: background (`KonvaImage`) + `AnnotationLayer` + in-progress drawing preview layer.

### Performance notes

- `PdfPage` and `AnnotationLayer` are wrapped in `React.memo`.
- `pageAnnotations` inside `PdfPage` is memoized with `useMemo` — gives `AnnotationLayer` a stable reference.
- Ctrl+scroll zoom is handled via a passive-false wheel listener on `window` in App.tsx (disabled in `fullscreen` and `grid` modes).
- `FullscreenView` manages its own zoom state (auto-fit on page change, Ctrl+scroll override). ESC calls `document.exitFullscreen()` first; `onExit()` is triggered by the resulting `fullscreenchange` event — this avoids the "window shrinks after React unmount" artefact.

### Bundle splitting & lazy loading

The renderer bundle is split so the initial chunk only contains code needed for the first paint. Everything optional is lazy-loaded on demand:

| Lazy chunk | Triggered by | Why it's lazy |
|---|---|---|
| `services/pdfExporter` | Export → PDF | pdf-lib (~200 KB) only needed at export time |
| `services/htmlExporter` | Export → HTML | base64 encoding helper |
| `services/imageExporter` | Export → Images | JSZip (~100 KB) only needed at export time |
| `services/pdfPageService` | Page CRUD ops | Shares pdf-lib with the PDF exporter |
| `components/modals/SignaturePad` | Editor → Signature | Canvas drawing UI |
| `components/modals/WatermarkConfig` | Editor → Watermark | Form UI |

`vite.config.ts` declares manual chunks for `react` / `react-dom` (~315 KB) and `konva` / `react-konva` (~181 KB) so they cache independently from app code across releases. Modals load through a `<Suspense fallback={null}>` boundary in `App.tsx`.

### Custom hooks (src/hooks/)

App.tsx is kept thin by extracting feature bundles into hooks. Each owns the state it needs, returns just the handlers, and lazy-imports any heavy dependencies internally.

| Hook | Returns | Notes |
|---|---|---|
| `useAnnotations()` | annotation state + CRUD + `remapAnnotations`, `clearMarkups` | Sole source of annotation state |
| `usePdfDocument(file)` | `{ pdfDoc, numPages, isLoading, error }` | Wraps pdfjs `getDocument` lifecycle |
| `useFitZoom({ pdfDoc, viewMode, rotation, setZoom })` | `{ calcFitZoom }` | Auto-fit on doc/view/rotation change (debounced 80 ms) |
| `usePrint()` | `{ handlePrint }` | Composites Konva canvases → image, listens to `wz-print` event |
| `useExporters({ ... onSuccess })` | `{ isExporting, handleExport{Pdf,Html,Images,Exe} }` | Lazy-imports services; surfaces a single `isExporting` for UI gating |
| `usePageOperations({ fileBytes, onResult })` | `{ isPageOperating, handle{Delete,InsertBlank,InsertFromPdf,Reorder}Pages }` | Shared `runOp` wrapper handles try/catch + state flips |
| `useThumbnails(pdfDoc, numPages)` | `thumbnails: string[]` (data URLs) | Sequential render at `THUMBNAIL_SCALE=0.2` |

### Page management

Page CRUD operations are handled by pure functions in `src/services/pdfPageService.ts` (uses **pdf-lib**). All functions return `{ newBytes: ArrayBuffer, pageMapping: Map<number, number> }` so callers can remap existing annotations via `useAnnotations().remapAnnotations(mapping)`.

| Function | Description |
|---|---|
| `deletePages(bytes, pageNums)` | Remove one or more pages (1-based); surviving pages are remapped |
| `insertBlankPage(bytes, afterPage, width?, height?)` | Insert a blank A4 page after `afterPage` (0 = prepend) |
| `insertPagesFromPdf(bytes, afterPage, srcBytes)` | Append all pages from another PDF after `afterPage` |
| `reorderPages(bytes, newOrder)` | Reorder pages according to a permutation array |

`useThumbnails(pdfDoc, numPages)` — hook that renders page thumbnails sequentially at `THUMBNAIL_SCALE=0.2` (JPEG quality 0.8). Returns `thumbnails: string[]` (data URLs). Re-runs whenever `pdfDoc` changes.

`PagePanel` (`src/components/panel/PagePanel.tsx`) — left sidebar showing thumbnails:
- Click to navigate, Ctrl/Shift-click for multi-select
- Drag-and-drop reorder (HTML5 drag API)
- Toolbar: Insert blank page, Insert from PDF, Delete selected pages
- `readOnly` prop: hides toolbar, disables drag, disables Delete key listener — used in Viewer mode
- Delete key shortcut: fires `handleDelete` when `!readOnly && selected.size > 0` and focus is not in an input/textarea

### Volatile markup tools (pen & rectangle)

Drawing tools that are **display-only and not exported to PDF**. They live alongside permanent annotations in the same `annotations` array but are identified by `isVolatile(a)` (`type === 'pen' || type === 'rectangle'`).

**Keyboard shortcuts (work in both normal and fullscreen mode):**

| Key | Action |
|---|---|
| `1` | Toggle yellow freehand pen mode (`activeMode = 'pen'`) |
| `2` | Toggle red rectangle drag mode (`activeMode = 'rectangle'`) |
| `ESC` (first press) | Clear all markups + exit drawing mode (if any markups or drawing mode active) |
| `ESC` (second press) | Exit fullscreen (if in fullscreen with no markups) |
| `Home` | Jump to first page (fullscreen only) |
| `End` | Jump to last page (fullscreen only) |

**Visual constants (in `PdfPage.tsx`):**

| Constant | Value | Purpose |
|---|---|---|
| `PEN_COLOR` | `'#FFFF00'` | Yellow pen stroke color |
| `PEN_STROKE_WIDTH` | `14` | PDF points (~21px at zoom=1) |
| `PEN_OPACITY` | `0.4` | Semi-transparent overlay feel |
| `RECT_COLOR` | `'#FF0000'` | Red rectangle stroke color |
| `RECT_STROKE_WIDTH` | `2` | PDF points |

**How continuous drawing works:**
- `addAnnotation` for volatile types does **not** reset `activeMode` or `selectedId`, so the user stays in drawing mode for multiple strokes.
- `handleAnnotationAdd` in App.tsx early-returns for pen/rectangle, skipping the `pendingStamp` clear and `setActiveMode('select')` path.
- The in-progress stroke is shown as a 3rd preview `<Layer listening={false}>` inside the Stage.

**Reset button:** Appears in ActionBar (hidden in fullscreen). Calls `clearMarkups()` + resets `activeMode` to `null`.

**Clearing markups:**
- `clearMarkups()` in `useAnnotations` removes all pen/rectangle annotations and preserves non-volatile `selectedId`.
- App.tsx global ESC handler runs in **capture phase** with `stopImmediatePropagation()` to prevent FullscreenView's bubble-phase handler from seeing the event when markups are being cleared.

### Annotation types

```typescript
// src/types/annotation.ts
type AnnotationType = 'stamp' | 'signature' | 'watermark' | 'pen' | 'rectangle'
type ActiveMode = 'select' | 'stamp' | 'signature' | 'watermark' | 'pen' | 'rectangle' | null

interface PenAnnotation extends BaseAnnotation {
  type: 'pen'
  points: number[]    // flat [x1,y1,x2,y2,...] in PDF points, page-local coords
  color: string       // '#FFFF00'
  strokeWidth: number // PDF points
  opacity: number     // 0–1
}

interface RectangleAnnotation extends BaseAnnotation {
  type: 'rectangle'
  color: string       // '#FF0000'
  strokeWidth: number // PDF points
}

const VOLATILE_TYPES: readonly AnnotationType[] = ['pen', 'rectangle']
function isVolatile(a: Annotation): boolean  // true for pen and rectangle
```

`remapAnnotations(mapping: Map<number, number>)` in `useAnnotations` — updates annotation page numbers after page CRUD. `allPages` watermarks are always preserved regardless of the mapping.

### Toast notifications

`src/components/Toast.tsx` — fixed bottom-center notification:
- Auto-dismisses after `duration` ms (default 2500ms)
- `role="status"` + `aria-live="polite"` for accessibility
- Rendered with a unique `key={toast.id}` in App.tsx so each new toast resets the timer even for identical messages
- Triggered by `showToast(message)` helper in App.tsx after successful export operations

### Export

Four export formats, all operating on `fileBytes` (not the rendered canvas). All show a toast on success.

| Format | Service | Notes |
|---|---|---|
| PDF (annotated) | `src/services/pdfExporter.ts` | Embeds stamp/signature/watermark using **pdf-lib**; pen/rectangle are volatile and **not** exported |
| HTML viewer | `src/services/htmlExporter.ts` | Self-contained file: PDF encoded as base64, decoded to a Blob URL at runtime |
| Images (ZIP) | `src/services/imageExporter.ts` | Each page rendered to PNG at 2× scale via pdfjs; bundled with **JSZip** |
| Viewer EXE | `electron/main.ts` `export-exe` IPC | Self-clone of the portable exe with PDF bytes appended; only works when running the packaged portable build |

### Viewer EXE feature

The current PDF can be exported as a standalone viewer exe in **both** the
portable run and the NSIS-installed app. The trick: a portable SFX template
is required as the base — the NSIS installer ships one in its resources.

`electron/main.ts` exposes `findViewerTemplate()` which resolves the template path:

1. **Portable run:** `process.env.PORTABLE_EXECUTABLE_FILE` is set by electron-builder
   and points to the running SFX itself. Use that.
2. **NSIS-installed run:** the env var is absent, but the installer bundled the
   portable as `<install>/resources/viewer-template.exe` (see afterPack hook below).
3. **Dev mode:** neither exists; `export-exe` IPC returns an error explaining the user must build first.

Export pipeline (regardless of source):

1. `findViewerTemplate()` resolves the template path.
2. The handler appends: `[PDF bytes] [4-byte length UInt32LE] [16-byte WZPDF_VIEWER_V01 marker]`.
3. On startup, `extractEmbeddedPdf()` checks `PORTABLE_EXECUTABLE_FILE` (the resulting EXE always runs as a portable SFX) for the marker and reads the embedded bytes.
4. If found, sends them to the renderer via the `open-pdf-bytes` IPC channel.

### Distribution (Windows)

`npm run build:exe` runs **two electron-builder invocations sequentially** —
`--win portable` then `--win nsis`. The order matters: NSIS's afterPack hook
needs the portable artifact already on disk to embed.

- `release/WZ_PDF_${version}.exe` — Portable single-file exe (~140 MB)
  - Built first
  - Acts as both a standalone launcher AND the template embedded by NSIS
  - `PORTABLE_EXECUTABLE_FILE` env-var is set automatically when launched

- `release/WZ_PDF_Setup_${version}.exe` — NSIS installer (~280 MB; recommended for daily use)
  - User chooses install path, creates Desktop + Start Menu shortcuts
  - Registers as a handler for `.pdf` files (`fileAssociations` in `electron-builder.json5`)
  - Maximum LZMA compression; only en-US/ko Electron locales bundled
  - **Bundles the portable** as `resources/viewer-template.exe` so the installed app can still produce Viewer EXE exports — see `scripts/afterPack.cjs`

The OS passes the double-clicked PDF path as a CLI argument; `electron/main.ts`
picks it up via `process.argv` and sends `open-file` to the renderer.

### Security hardening (electron/main.ts)

Renderer is sandboxed and IPC inputs are validated. Notable measures:

- `BrowserWindow.webPreferences`: `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`, `webSecurity: true`, `allowRunningInsecureContent: false`, `experimentalFeatures: false`, plus the `*InWorker`/`*InSubFrames` variants.
- `will-navigate` blocks all URLs except `file://` (production) and `http://localhost:5173` (dev).
- `setWindowOpenHandler` denies new Electron windows; external `http(s)://` URLs are handed to `shell.openExternal`.
- `web-contents-created` denies `<webview>` attachments app-wide.
- **Production-only CSP** is injected via `session.defaultSession.webRequest.onHeadersReceived`. Dev mode skips this (Vite HMR needs `unsafe-eval` and a WebSocket connect, which would weaken the policy).
- `read-file` IPC validates the path: non-empty string, resolves to a real file, `.pdf` extension, size ≤ 500 MB. Even though the path normally comes from the OS (CLI arg / open-file event), defense-in-depth assumes the renderer could be compromised.

## Critical gotchas

### pdfjs polyfills (must stay in `src/main.tsx`)
Both polyfills — `Uint8Array.prototype.toHex` and `Map.prototype.getOrInsertComputed` — must appear in **two places**: the main thread (before pdfjs imports) and inside the blob worker string. Removing either will break PDF rendering in Electron.

### Annotation coordinates
Everything stored and exported uses the `effectiveZoom = PDF_RENDER_SCALE * zoom` divisor. If you pass plain `zoom` instead of `effectiveZoom` to `toStoredCoords`, annotations will be placed at the wrong position relative to the PDF.

### `page.render()` API (pdfjs 5.x)
Call as `page.render({ canvas, viewport })` — NOT `{ canvasContext: ctx, viewport }`. The old API was removed in pdfjs 5.x.

### Fullscreen two-step exit (ESC priority)
The two-step ESC behavior ("first press clears markups, second press exits fullscreen") requires:
1. **Keyboard Lock API** in FullscreenView: `navigator.keyboard.lock(['Escape'])` prevents the browser/OS from auto-exiting fullscreen before any JS handler fires.
2. **Capture-phase listener** in App.tsx: `window.addEventListener('keydown', handler, true)` fires before FullscreenView's bubble-phase listener.
3. **`stopImmediatePropagation()`** in App.tsx ESC handler: blocks FullscreenView's handler from also triggering `exitFullscreen()` on the same keydown when clearing markups.

Do **not** call `onExit()` directly from an ESC keydown handler. Always call `document.exitFullscreen()` first and let the `fullscreenchange` event trigger `onExit()`.

### Window-level mouseup for spread-mode drawing
In spread (two-page) view, dragging a pen stroke across the boundary between two `<Stage>` elements means the originating Stage never receives `mouseup`, leaving the stroke orphaned. Fix: each `PdfPage` registers a `window.addEventListener('mouseup', commitDraft)` safety net while in drawing mode. `commitDraft` is wrapped in `useCallback` with a `draftRef.current = null` guard so it is idempotent — calling it twice (once from `window`, once from Stage) is safe.

### Hooks must precede early returns in PdfPage
`PdfPage` has an early return for the loading skeleton (`if (isLoading || !pageData) return …`). All React hooks — including the drawing state (`draft`/`draftRef`), `commitDraft`, and the window-level `mouseup` useEffect — must be declared **before** this early return, or React will throw a "hooks called in different order" error.

### addAnnotation volatile behavior
`addAnnotation` in `useAnnotations` checks `annotation.type === 'pen' || 'rectangle'` and, for these types, **does not** reset `activeMode` or `selectedId`. This allows continuous multi-stroke drawing without leaving drawing mode. The corresponding `handleAnnotationAdd` in App.tsx early-returns for volatile types so `pendingStamp`/`pendingSignature` state is not cleared and `setActiveMode('select')` is not called.

### Electron native print timing
Since Electron's `webContents.print` is asynchronous, do not restore the DOM canvases (`afterPrint` cleanup) immediately after starting the print. Instead, expose the print call as a Promise and `await` it in the renderer, so cleanup is deferred until the print dialog is closed.

### TypeScript Omit on union types
TypeScript's `Omit<T, K>` does not distribute over union types (it resolves to common keys first, stripping unique properties from union members). Use a distributed utility:
`type OmitId<T> = T extends any ? Omit<T, 'id'> : never`

### Viewer EXE export — template resolution
The `export-exe` IPC handler can't work in dev mode because no SFX template exists. In portable runs it uses `PORTABLE_EXECUTABLE_FILE`; in NSIS-installed runs it uses the bundled `resources/viewer-template.exe`. See `findViewerTemplate()` in `electron/main.ts`. To test the feature you must run a packaged build (either artifact).

### build:exe sequencing — portable MUST build first
The script in `package.json` runs `electron-builder --win portable` *and then* `electron-builder --win nsis` deliberately, not as one combined invocation. The NSIS `afterPack` hook (`scripts/afterPack.cjs`) embeds the portable artifact as `viewer-template.exe`, which requires the portable to already exist on disk. A combined invocation would share `win-unpacked/` and the template wouldn't be ready when NSIS packs.

### Claude Code file locks during build
The `claude.exe` agent process can hold open file handles to `release/win-unpacked/resources/app.asar` from previous Glob/Read tool calls, causing electron-builder to fail with "process cannot access the file because it is being used by another process". Before a `build:exe` run, either restart the Claude Code session or delete `release/` from a separate admin terminal. Also add the project folder to Windows Defender exclusions if real-time scanning is locking newly written asars.

---
> Source: [whyzoo-lab/WZ-PDF](https://github.com/whyzoo-lab/WZ-PDF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
