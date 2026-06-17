## hvac-helper

> - **Framework:** React with Vite

# Project Instructions

- **Framework:** React with Vite
- **Styling:** Tailwind CSS only. No custom CSS.
- **Tone:** Be concise. Don't explain basic React concepts. Do explain non-obvious architectural decisions since the owner is not an experienced frontend developer.
- **Language:** JavaScript only, no TypeScript
- **Dev OS:** Windows, PowerShell. Use `;` instead of `&&` for chaining commands.
- **Code style:** Modular, readable, broken into small components. Other developers (including non-experts) need to be able to understand this code.

---

## What This App Is

**HVAC Helper** is a local-first, browser-based floor plan annotation tool for HVAC professionals. It is an alternative to plandroid.com.

The user loads a PDF floor plan, draws labeled sections/zones on top of it (rectangles or polygons), and exports a new PDF with those annotations burned in, pixel-perfectly aligned.

---

## Current Features (Do Not Break These)

- Load and view multi-page PDFs
- Auto-scale PDF to fit the workspace
- Page-by-page navigation
- Draw rectangles (click and drag)
- Draw custom polygons (click to place nodes, click start node to close)
- Move and resize any shape via bounding box (Konva Transformer)
- Sidebar: list, rename, delete sections
- Sections are page-aware — each shape belongs to a specific page
- Auto color-coding for shapes
- Export: original PDF + all annotations overlaid, with correct coordinates
- **Rotation-aware export:** handles PDFs with internal rotation metadata (0°, 90°, 180°, 270°). Annotations always land where they were drawn visually regardless of PDF rotation.

---

## Planned Features (Design With These In Mind)

Do not implement these now, but make architectural decisions that won't block them later:

- Ductwork drawing tools (lines with width, bends)
- Load calculations based on section area/volume
- Equipment placement (drag-and-drop symbols: units, vents, thermostats)
- Bill of Materials (BOM) auto-generation from drawn plan
- Cloud sync / team sharing

This means: keep shape data extensible, keep drawing tools pluggable, keep the sidebar generic enough to show different shape types.

---

## Approved Package Versions

Always use these exact versions. Do not hallucinate versions that do not exist.

```json
"dependencies": {
  "konva": "^10.2.3",
  "lucide-react": "^0.383.0",
  "pdf-lib": "^1.17.1",
  "pdfjs-dist": "^4.4.168",
  "react": "^19.0.0",
  "react-dom": "^19.0.0",
  "react-konva": "^19.0.0",
  "uuid": "^11.0.0"
}
```

**Do NOT use `react-pdf`.** It conflicts with the Konva layer approach. Use `pdfjs-dist` directly for rendering.

---

## Architecture

Three completely separate concerns. Never mix them.

```
┌──────────────────────────────────┐
│         react-konva Stage        │  drawing, selection, interaction
├──────────────────────────────────┤
│    pdfjs-dist → canvas → image   │  PDF display only
├──────────────────────────────────┤
│            pdf-lib               │  export only, never for display
└──────────────────────────────────┘
```

### PDF Display (pdfjs-dist)
- Render the PDF page to a hidden `<canvas>` using pdfjs-dist
- Convert that canvas to a data URL via `canvas.toDataURL()`
- Render that data URL as a Konva `<Image>` in the background layer of the Stage
- This gives a static visual of the PDF that Konva can sit on top of

```
<div style={{ position: 'relative' }}>
  <canvas style={{ display: 'none' }} />     // pdfjs renders here
  <Stage>
    <Layer>
      <Image />                              // PDF as background
    </Layer>
    <Layer>
      <Rect /> / <Line />                    // user drawn shapes
      <Text />                               // shape labels
      <Transformer />                        // selection/resize handles
    </Layer>
  </Stage>
</div>
```

### Shape Drawing (react-konva)
- Rectangles: mousedown → mousemove → mouseup to define bounds
- Polygons: click to place each node, click on first node to close path, use Konva `<Line closed />`
- After finishing a shape, prompt user for a name
- All shapes stored in React state

### PDF Export (pdf-lib)
- Load original PDF bytes into pdf-lib
- For each shape on each page, convert coordinates (see below) and draw onto the pdf-lib page
- ALWAYS account for PDF page rotation when exporting (see rotation section below)
- Save and trigger a browser download

---

## Coordinate System — This Is The Most Critical Part

PDF coordinate space and Konva/screen coordinate space are completely different. Getting this wrong means shapes appear in the wrong place on export. Never pass raw Konva coordinates to pdf-lib.

| | Screen / Konva | PDF (pdf-lib) |
|---|---|---|
| Origin | Top-left | Bottom-left |
| Y direction | Increases downward | Increases upward |
| Units | Pixels | Points |

### Rectangle Conversion

```js
// stageWidth/stageHeight = the rendered Konva stage size in pixels
// pdfWidth/pdfHeight = the PDF page size in points from pdf-lib page.getSize()

function rectToPdfCoords(shape, stageWidth, stageHeight, pdfWidth, pdfHeight) {
  const scaleX = pdfWidth / stageWidth;
  const scaleY = pdfHeight / stageHeight;

  return {
    x: shape.x * scaleX,
    y: pdfHeight - (shape.y + shape.height) * scaleY,  // flip Y axis
    width: shape.width * scaleX,
    height: shape.height * scaleY,
  };
}
```

### Polygon Conversion

Polygons are stored as a flat array of points `[x1, y1, x2, y2, ...]` in Konva pixel space.

```js
function polygonToPdfCoords(points, stageWidth, stageHeight, pdfWidth, pdfHeight) {
  const scaleX = pdfWidth / stageWidth;
  const scaleY = pdfHeight / stageHeight;

  const converted = [];
  for (let i = 0; i < points.length; i += 2) {
    converted.push(points[i] * scaleX);                   // x
    converted.push(pdfHeight - points[i + 1] * scaleY);   // y flipped
  }
  return converted;
}
```

To draw a polygon in pdf-lib, convert the points into an SVG path string and use `page.drawSvgPath()`.

---

## PDF Rotation Handling — Already Implemented, Do Not Break

Some PDFs store pages with internal rotation metadata (0°, 90°, 180°, 270°). The viewer displays them upright, but the coordinate space is rotated internally. Export must compensate for this or annotations will be misaligned.

When exporting with pdf-lib, always read the page rotation:

```js
const rotation = page.getRotation().angle; // 0, 90, 180, or 270
const { width: pdfWidth, height: pdfHeight } = page.getSize();
```

The existing implementation already handles rotation compensation correctly. Do not remove, simplify, or rewrite this logic. If modifying the export function for any reason, test with a rotated PDF before considering it done.

---

## Shape Data Model

Every shape regardless of type must have these base fields:

```js
{
  id: "uuid",           // uuid()
  type: "rect",         // "rect" | "polygon"
  page: 1,              // 1-indexed page number
  name: "Zone A",       // user provided label
  color: "#e74c3c",     // stroke color
}
```

Type-specific fields:

```js
// Rectangle
{
  ...base,
  type: "rect",
  x: 120,
  y: 80,
  width: 200,
  height: 150,
}

// Polygon
{
  ...base,
  type: "polygon",
  points: [x1, y1, x2, y2, x3, y3, ...],  // flat array, Konva format
}
```

Keep this extensible — future shape types (lines for ductwork, symbols for equipment) will follow the same base pattern.

---

## State Structure

```js
{
  pdfFile: null,          // original File object
  pdfBytes: null,         // ArrayBuffer of original PDF, kept for export
  totalPages: 1,
  currentPage: 1,
  pageImage: null,        // data URL of rendered current page
  stageWidth: 0,          // rendered stage width in pixels
  stageHeight: 0,         // rendered stage height in pixels
  shapes: [],             // all shapes across all pages
  selectedId: null,       // currently selected shape id
  tool: "rect",           // active drawing tool: "rect" | "polygon" | "select"
}
```

---

## Key Implementation Details

### pdfjs Worker Setup

Always use the CDN URL matching the installed version. Do not bundle the worker locally — it causes issues with Vite.

```js
import * as pdfjsLib from 'pdfjs-dist';
pdfjsLib.GlobalWorkerOptions.workerSrc =
  'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.4.168/pdf.worker.min.js';
```

### Loading and Rendering a Page

```js
const loadPage = async (pdf, pageNumber) => {
  const page = await pdf.getPage(pageNumber);
  const viewport = page.getViewport({ scale: 1.5 });

  const canvas = document.createElement('canvas');
  canvas.width = viewport.width;
  canvas.height = viewport.height;

  await page.render({
    canvasContext: canvas.getContext('2d'),
    viewport,
  }).promise;

  return {
    imageDataUrl: canvas.toDataURL(),
    width: viewport.width,
    height: viewport.height,
  };
};
```

### Export

- Load original PDF bytes (kept in state as ArrayBuffer) into pdf-lib
- Iterate shapes filtered by page number
- Apply coordinate conversion per shape type (rect vs polygon)
- Apply rotation compensation
- Draw using pdf-lib's `drawRectangle()` for rects, `drawSvgPath()` for polygons
- Draw text label near each shape
- Save and trigger download via blob URL

---

## Folder Structure

```
src/
  App.jsx
  main.jsx
  components/
    PdfUploader.jsx        # file input and drag-and-drop
    PdfViewer.jsx          # pdfjs rendering, page navigation
    AnnotationCanvas.jsx   # Konva stage, drawing interaction, shape rendering
    ShapeList.jsx          # sidebar: list, rename, delete shapes
    Toolbar.jsx            # tool selector, page nav, export button
  hooks/
    usePdfLoader.js        # pdfjs loading and page rendering logic
    useShapes.js           # shape CRUD, color assignment
    useDrawing.js          # in-progress drawing state (mousedown → mouseup)
  utils/
    coordinates.js         # ALL coordinate conversion functions live here
    pdfExport.js           # pdf-lib export logic, imports from coordinates.js
    colors.js              # auto color cycling for new shapes
```

---

## Rules

- No TypeScript
- No custom CSS — Tailwind only
- Do NOT use `react-pdf` — use `pdfjs-dist` directly
- Never pass raw Konva coordinates to pdf-lib — always go through `coordinates.js`
- Never hardcode pdfjs worker as a local file — use the CDN URL
- Use `npm install`, not yarn or pnpm
- PowerShell only — use `;` to chain commands, not `&&`
- Keep components small and single-purpose
- Comments should explain WHY, not WHAT

---
> Source: [Arpitpatel1771/hvac-helper](https://github.com/Arpitpatel1771/hvac-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
