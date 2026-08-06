## lattica

> This file tells coding agents (Claude Code, Cursor, etc.) how to use **Taible**

# AGENTS.md — Using Taible (for AI coding agents)

This file tells coding agents (Claude Code, Cursor, etc.) how to use **Taible**
correctly. It is task-oriented and copy-paste friendly. For human prose see
`docs/USAGE.md`; for the full feature log see `docs/PROGRESS.md`.

## What Taible is

A clean-room, **MIT-licensed** data grid + spreadsheet engine for **React /
Next.js**. No GPL/Handsontable/HyperFormula code. Canvas rendering + DOM editing
hybrid, a self-built Excel-compatible formula engine (150 functions), CRDT
collaboration, AI-native helpers, and an MCP tool layer.

Monorepo (pnpm). Eight published packages:

| Package | Import from | Purpose |
|---|---|---|
| `@ai-path/tb-core` | `@ai-path/tb-core` | Headless models: sizes, selection, undo, merge, validation, conditional format (value + visual), number format, aggregate, pivot, sparkline, chart layout, detail, fill, coords. No React/DOM. |
| `@ai-path/tb-formula` | `@ai-path/tb-formula` | `SheetEngine`, parser, 150 functions, dependency graph, spill, named ranges, R1C1, structured refs. |
| `@ai-path/tb-data` | `@ai-path/tb-data` | Visual↔physical index mapping, sort/filter models, nested rows, `DataView`, `AsyncRowModel`. |
| `@ai-path/tb-react` | `@ai-path/tb-react` | `<LatticaGrid>`, `<LatticaStatusBar>`, `<LatticaChart>`, `useGridController`, themes/palette/density. |
| `@ai-path/tb-io` | `@ai-path/tb-io` | CSV/TSV, XLSX read + plain/`writeStyledXlsx`, JSON, clipboard, `tableToPdf`. |
| `@ai-path/tb-collab` | `@ai-path/tb-collab` | CRDT (LWW), fractional index keys, presence, transport. |
| `@ai-path/tb-ai` | `@ai-path/tb-ai` | Provider-agnostic NL→formula/operation, smart fill, anomaly, etc. (MockProvider for tests). |
| `@ai-path/tb-mcp` | `@ai-path/tb-mcp` | Grid tool registry + `ToolDispatcher` for AI agents. |

## Install

```bash
pnpm add @ai-path/tb-react @ai-path/tb-core @ai-path/tb-formula
# add @ai-path/tb-io @ai-path/tb-data @ai-path/tb-ai @ai-path/tb-mcp @ai-path/tb-collab as needed
```

Peer deps: `react`/`react-dom` ≥ 18 (tested on 19). ESM + CJS are both shipped.

> **Import rule:** consumers import from the package name (`@ai-path/tb-react`).
> The `.js` suffix you see on *internal* relative imports (`./foo.js`) is a
> source convention for NodeNext ESM — do **not** add it to package imports.

## Quickstart — a React grid

```tsx
'use client';
import { useState } from 'react';
import { LatticaGrid, useGridController } from '@ai-path/tb-react';
import type { ColumnNode } from '@ai-path/tb-core';

const columns: ColumnNode[] = [
  { headerName: 'Item', field: 'item', width: 180, type: 'text' },
  { headerName: 'Qty', field: 'qty', width: 80, type: 'number', align: 'right' },
  { headerName: 'Total', field: 'total', width: 120, type: 'number', format: '#,##0' },
];

export default function Demo() {
  const controller = useGridController({ rowCount: 1, colCount: 1 });
  const [rows] = useState([
    { item: 'Apple', qty: 3, total: '=B1*100' },
    { item: 'Pear', qty: 2, total: '=B2*120' },
  ]);
  return <LatticaGrid controller={controller} rows={rows} columns={columns} width={800} height={480} />;
}
```

- `useGridController(options)` returns a stable headless `GridController`.
- `<LatticaGrid controller rows columns width height autoSize maxWidth maxHeight fill theme renderDetail contextMenu onCellCommit editSelection cellOverlay renderCellOverlay onCellOverlayClose />`.
- `columns` are optional multi-level header defs (`ColumnNode` = leaf `{headerName}`
  or group `{headerName, children, collapsible?, showWhen?}`). Leaf defs may carry
  rich metadata such as `field`, `width`, `type`, `editable`, `align`, `format`,
  `options`, and `maxLength`. Omit for A,B,C… letters.
- `rows` is controlled record data. Leaf `field` values define the extraction order;
  a fieldless leaf receives empty cells. When `rows` changes, the controller resizes
  and replaces grid data without adding undo history.
- The imperative API remains available for edits and programmatic writes:
  `controller.setCellText(0, 0, 'Apple')`, `controller.setData(matrix)`, and
  `controller.setRecords(records, fields)`.
- `autoSize="content"` sizes the grid to visible content
  (`rowHeaderWidth + visible column widths`, `colHeaderHeight + visible row heights`).
  `maxWidth` / `maxHeight` clamp that size and leave overflow scrollable. When
  `autoSize` is set, `width`, `height`, and `fill` are ignored.

## Public contract (stable identifiers)

The following DOM hooks are public and stable. Renaming or changing their
semantics is a breaking change:

- `data-testid="lattica-grid"`
- `data-testid="lattica-editor"`, `lattica-editor-select`,
  `lattica-editor-date`, `lattica-editor-autocomplete`, `lattica-editor-datalist`
- `data-testid="lattica-filter-<col>"`, `lattica-sort-<col>`
- `data-testid="lattica-cell-overlay"`
- `data-testid="lattica-colsettings"`, `lattica-colsettings-vis-<physicalCol>`,
  `lattica-colsettings-width-<physicalCol>`, `lattica-colsettings-showall`,
  `lattica-colsettings-resetwidths`
- `data-testid="lattica-static-table"`
- `data-testid="lattica-rowgroup-<row>"`

ARIA labels are also stable for interactive chrome:
`filter column N`, `sort column N`, `toggle row group N`, and column-settings
visibility labels (`Show <column>`). Treat changes to these strings as breaking.

## The coordinate model (important)

The grid distinguishes **visual** (what the user sees, after sort/filter/move/hide)
from **physical** (storage) indices. Public `GridController` methods take **visual**
indices and map internally. When you need the data-row behind a visual row, use
`controller.getPhysicalRow(visualRow)`. Formulas operate on **physical** A1 cells.

## GridController — the headless API (most-used)

```ts
// content
setCellText(row, col, raw)            // raw: '=formula' | number-ish string | text
getDisplay(row, col): string          // formatted value
getValue(row, col): unknown           // raw value (error → '#DIV/0!' string)
getEditText(row, col): string         // original input (=formula or literal)
getRowCount() / getColCount()
setRowCount(n) / setColCount(n)       // resize view; engine content outside range is retained
setData(matrix, { resize? })          // full data replacement, no undo history
setRecords(records, fields, opts)     // record → matrix helper used by <LatticaGrid rows>

// selection / edit / clipboard / undo
selection.setActive({row,col}); selection.extendTo({row,col})
beginEdit/updateDraft/commitEdit/cancelEdit
on('cellcommit', e => ...)             // source + raw prev/next cell changes
copySelection(): string[][]; paste(matrix); deleteSelection()
undoLast(); redoLast()

// columns
setColumnType(col, 'text'|'number'|'checkbox'|'dropdown'|'date'|'autocomplete'|'time')
setColumnAlign(col, 'left'|'center'|'right')
setColumnOptions(col, string[])        // dropdown/autocomplete + list validator
setColumnFormat(col, '#,##0.00')       // Excel number format
setColumnValidator(col, v => boolean)  // invalid cells tint red
setColumnEditable(visualCol, boolean); setCellReadOnly(row, col, boolean); isCellEditable(row, col)
setColumnInput(visualCol, { sanitizeDraft?, maxLength?, commitTransform? } | null)
hideColumn(visualCol); showColumn(physicalCol); showAllColumns()
setColumnVisible(physicalCol, visible); setColumnWidth(physicalCol, width); resetColumnWidths()
moveColumn(fromVisual, toVisual)

// sort / filter / find
toggleSort(col, additive?)             // none→asc→desc→none
setColumnFilter(col, conditions[])     // {kind:'gt',value} | {kind:'equals',value} | 'in' | 'contains' …
columnFacets(col); setColumnSetFilter(col, values[]) // Excel-style value picker
clearView()
replaceAll(query, replacement, {regex?,caseSensitive?,wholeCell?}): number

// aggregation / status bar
aggregateColumn(col, 'sum'|'avg'|'count'|'min'|'max'|'median'): number|null
selectionSummary(): { count, sum, avg, min, max }

// visual conditional formatting
setColorScale(col, ['#fee','#fca','#16a34a'])
setDataBar(col, '#93c5fd')
setIconSet(col, ['🔴','🟡','🟢'])
conditionalFormat.addRule({ kind:'gt', value:70, style:{ background:'#d8f5d0', bold:true } })

// merge / nested rows / sparkline / master-detail
mergeSelection(); unmerge()
setRowTree(nodes); toggleRowGroup(row)
setCellSparkline(row, col, number[], 'line'|'bar'|'winloss')
toggleDetail(row); setDetailHeight(px)  // render with <LatticaGrid renderDetail={fn}>

// fill handle
fillTo(row, col)
```

The underlying formula engine is `controller.engine` (a `SheetEngine`).

## Formula engine headless (no React)

```ts
import { SheetEngine } from '@ai-path/tb-formula';
const e = new SheetEngine();
e.setContent({ row: 0, col: 0 }, 10);
e.setContent({ row: 0, col: 1 }, '=A1*2');
e.getValue({ row: 0, col: 1 }); // 20
e.evaluateFormula('=SUM(SEQUENCE(3))'); // one-off
e.defineName('Tax', '=0.1'); e.defineTable('Sales', { row:1, col:0, rowCount:100, headers:['Item','Amount'] });
```

Supported: 150 functions (math/stat/text/date/financial/logical/lookup incl.
`XLOOKUP`/`XMATCH`, dynamic arrays `FILTER`/`SORT`/`SORTBY`/`UNIQUE`/`SEQUENCE`/
`TRANSPOSE`/`VSTACK`/`HSTACK`, `LET`, `LAMBDA`+`MAP`/`REDUCE`/`SCAN`/`BYROW`/`BYCOL`),
**spill** (anchor shows top-left, fills neighbors, `#SPILL!` on collision),
**named ranges**, **structured references** `Table[Col]`, R1C1. `builtinFunctionNames()`
lists them all.

## Recipes

**Rich editors + validation** — set the column type; dropdown/autocomplete also take options:
```ts
controller.setColumnType(0, 'dropdown'); controller.setColumnOptions(0, ['A','B']);
controller.setColumnType(1, 'date');
controller.setColumnValidator(2, v => typeof v === 'number' && v > 0);
```

**Edit lifecycle and input control** — subscribe to committed writes, restrict UI editing, or normalize drafts:
```tsx
<LatticaGrid controller={controller} onCellCommit={(e) => saveAudit(e)} editSelection="end" />
controller.setColumnEditable(0, false);        // UI beginEdit is blocked
controller.setCellReadOnly(2, 1, true);        // cell-level read-only wins
controller.setColumnInput(1, {
  sanitizeDraft: (draft) => draft.replace(/\D/g, ''),
  maxLength: 8,
  commitTransform: (raw) => raw === '' ? null : raw,
});
controller.setColumnType(2, 'time');           // accepts 930/1330/9:30 and stores HH:mm
```

**Cell-anchored overlays** — use the public ref handle for viewport rects, or the controlled overlay API for React popovers anchored inside the grid:
```tsx
import { useRef, useState } from 'react';
import { LatticaGrid, type LatticaGridHandle } from '@ai-path/tb-react';

const gridRef = useRef<LatticaGridHandle>(null);
const [overlay, setOverlay] = useState<{ row: number; col: number } | null>(null);

<LatticaGrid
  ref={gridRef}
  controller={controller}
  cellOverlay={overlay}
  onCellClick={(cell) => setOverlay(cell)}
  onCellOverlayClose={() => setOverlay(null)}
  renderCellOverlay={({ row, col, rect, close }) => (
    <div style={{ minWidth: rect.width }}>
      {controller.getDisplay(row, col)}
      <button type="button" onClick={close}>Close</button>
    </div>
  )}
/>;

gridRef.current?.scrollCellIntoView(20, 3);
const viewportRect = gridRef.current?.getCellClientRect(20, 3);
```

**Faceted filter / hide / move** — UI is built in (`▽` header button, header context menu),
or drive headlessly via `setColumnSetFilter` / `hideColumn` / `moveColumn`.

**Pivot table**
```ts
import { pivot, pivotToMatrix } from '@ai-path/tb-core';
const r = pivot(records, { rows:['region'], columns:['product'], value:'units', agg:'sum' });
const matrix = pivotToMatrix(r); // header + body + totals; write into a grid
```

**Charts & sparklines**
```tsx
import { LatticaChart } from '@ai-path/tb-react';
<LatticaChart spec={{ kind:'bar', categories:['Q1','Q2'], series:[{name:'N', values:[3,5]}] }} width={320} height={200} />
controller.setCellSparkline(0, 1, [3,5,4,7], 'line');
```

**Themes & density**
```ts
import { buildTheme, densityMetrics, densityOptions } from '@ai-path/tb-react';
const theme = buildTheme({ palette: 'midnight', density: 'spacious' });
const editTheme = buildTheme({ overrides: { readOnlyCellBackground: '#f4f4f5' } });
const c = useGridController({ rowCount: 100, colCount: 8, ...densityOptions('compact') });
const compactMetrics = densityMetrics('compact'); // row/col/header dimensions for surrounding UI
<LatticaGrid controller={c} theme={theme} />
<LatticaStatusBar controller={c} theme={theme} />
```
Palettes: `light dark highContrast midnight sepia solarizedLight solarizedDark`.
Densities: `compact comfortable spacious`. `buildTheme({ palette, density, fontFamily, overrides })`.

**Content-sized grid**
```tsx
<LatticaGrid controller={controller} rows={rows} columns={columns} autoSize="content" maxHeight={320} />
```
`autoSize="content"` follows controller `change` events such as row/column count,
row height, column width, filtering, hiding, and detail expansion changes.

**Column settings panel**
```tsx
import { LatticaColumnSettings } from '@ai-path/tb-react';

<LatticaColumnSettings
  controller={controller}
  columns={columns}
  showVisibility
  showWidths={isAdmin}
  title="Columns"
/>
```

`<LatticaColumnSettings>` lists physical columns in the current visual order and
lets users toggle visibility. Admin-style UIs can pass `showWidths` to expose
number inputs for physical column widths, plus a reset button. Props:
`controller` (required), `columns`, `theme`, `showVisibility` (default `true`),
`showWidths` (default `false`), and `title` (default `Columns`).

**View-state persistence (per-user / org-wide)**
```tsx
import { deserializeState, serializeState } from '@ai-path/tb-core';
const controller = useGridController({ rowCount: 1000, colCount: 20 });
useEffect(() => {
  if (orgDefaultJson) controller.applyViewState(deserializeState(orgDefaultJson));
  const saved = localStorage.getItem(`lattica:view:${userId}`);
  if (saved) controller.applyViewState(deserializeState(saved));
}, [controller, orgDefaultJson, userId]);
<LatticaGrid
  controller={controller}
  onViewStateChange={(s) => localStorage.setItem(`lattica:view:${userId}`, serializeState(s))}
/>
```

**Export**
```ts
import { serializeDelimited, matrixToXlsx, writeStyledXlsx, tableToPdf } from '@ai-path/tb-io';
serializeDelimited(rows);                 // CSV (string)
matrixToXlsx(rows, 'Sheet1');             // Uint8Array (values)
writeStyledXlsx({ sheets:[{ name:'S', rows: styledCells, merges }] }); // numFmt/fills/bold/merge
tableToPdf(rows, { title:'Report' });     // Uint8Array (PDF, Latin-1)
```

**Print / SSR static table**
```tsx
import { renderStaticTable, staticTablePrintCss } from '@ai-path/tb-react';

<style>{staticTablePrintCss}</style>
{renderStaticTable(controller, columns, {
  includeRowNumbers: true,
  maxRows: 500,
  caption: 'Printable view',
})}
```
`renderStaticTable` reads the current controller view (sort/filter/hide/move,
column widths, displayed values, and column alignment) and returns a plain React
`<table>` element for `renderToStaticMarkup`, SSR, or browser printing.

**Async / server-side rows**
```ts
import { AsyncRowModel } from '@ai-path/tb-data';
const m = new AsyncRowModel({ blockSize: 50, fetcher: (offset, limit) => fetchPage(offset, limit) });
await m.ensureRange(start, end); m.getRow(i); m.getTotal(); m.subscribe(rerender);
```

**AI (provider-agnostic)** — pass any provider; tests use `MockProvider`.
**MCP** — `createGridTools(engine)` + `ToolDispatcher` expose get/set/range/evaluate/define_name.

## Conventions & gotchas

- **Headless first.** All logic lives in `@ai-path/tb-core`/`@ai-path/tb-data`/`@ai-path/tb-formula`;
  React is a thin view. Prefer driving the `GridController`, not DOM.
- **Visual vs physical** indices (see above). Don't pass physical indices to public
  controller methods or vice-versa.
- **Spilled cells are virtual** — they have no stored content; writing into one blocks
  the spill (`#SPILL!`).
- **Errors are values** — `getValue` returns `'#DIV/0!'`-style strings; the engine
  returns `FormulaError` objects (use `FormulaError.is(x)`).
- **No runtime deps** in core/io (self-built ZIP/deflate/PDF). Keep it that way.
- **PDF** uses Helvetica/WinAnsi; non-Latin-1 chars become `?` (no font embedding).

## SSR/Next.js: dynamic import not required (server-safe import guaranteed)

`@ai-path/tb-react` must import and server-render in Next.js App Router without `next/dynamic({ ssr: false })`; keep browser globals in effects/event handlers or behind `typeof` guards, and preserve the node-environment SSR test.

## Manufacturing-report features (tayca batch, 2026-07-05)

Ten additive features shipped for the Handsontable-replacement requirements in
`docs/PRODUCT_REQUIREMENTS_TAYCA.md` (PRs #58–#68). All are opt-in and
backward compatible.

### Time & elapsed input (`time` options, `elapsed` type)

```ts
controller.setColumnType(2, 'time');                               // 930 / 1613 / 635 / 18 → HH:MM (range-checked)
controller.setColumnType(3, 'time', { timeOrDecimalHours: true }); // "1.5" stays a decimal-hours number
controller.setColumnType(4, 'time', { excelTimeSerial: true });    // "0.5" (0<x<1) → "12:00"
controller.setColumnType(5, 'elapsed');                            // "30:15" stored as-is, displayed "1:06:15"
controller.getColumnTypeOptions(3);                                // → { timeOrDecimalHours: true }
```

Pure helpers: `normalizeFlexibleTimeInput`, `excelTimeSerialToHhMm`,
`parseElapsedTime`, `normalizeElapsedInput`, `formatElapsedDisplay`.

### Full-width input policy + reject event

`number`/`time` columns normalize zenkaku digits (０-９ ．，－−＋：) by default.

```tsx
<LatticaGrid onInputReject={(e) => toast(`${e.reason}: ${e.raw}`)} ... />
// e: { row, col, raw, reason: 'fullwidth' | 'transform' | 'validator' }
```

```ts
controller.setColumnFullWidthMode(1, 'reject');  // 'reject' | 'normalize' | 'off' | null(=default)
// ColumnDef: { field: 'qty', type: 'number', fullWidthMode: 'reject' }
```

Cleared cells always commit as empty (stored `null`, never `""`); numeric `0`
renders through the column format (`"0.00"`), never blank.

### Multi-line header labels & unit-tier merge

`headerName` may contain `\n` (renders multi-line; header band auto-grows).
A non-collapsible group whose only child is a `headerName: ''` leaf is
absorbed: the parent label spans down to the bottom header row (3-tier
group/name/unit layouts with unit-less columns).

```ts
controller.getHeaderHeight();      // effective band height (px) — align external strips with this
controller.getBaseHeaderHeight();  // pre-expansion base
// Theme tokens: headerLineHeight (16), headerPaddingY (3)
```

### Display-only override

```tsx
<LatticaGrid displayValue={(row, col, base) => auditPreview?.get(`${row}:${col}`) ?? null} ... />
// or controller.setDisplayOverride(fn) / setDisplayOverride(null) to clear
```

Painted text only — stored values, edit text, and copy output are untouched.

### Navigation / selection / undo / context-menu config

```tsx
<LatticaGrid
  enterMoves={{ row: 0, col: 1 }}   // Enter commits then moves right
  enterBeginsEditing={false}
  tabNavigation={false}
  outsideClickDeselects={false}
  selectionDisabled                  // view-only: no selection visuals/interaction
  contextMenu="clipboard-only"       // 'none' | 'clipboard-only' | 'full' | (target) => MenuItemSpec[]
  ...
/>
```

`controller.undo.setEnabled(false)` pauses history (push/undo/redo become
no-ops; `canUndo()`/`canRedo()` report false) — re-enable with `setEnabled(true)`.

### Word wrap + row auto-sizing

```ts
// ColumnDef: { field: 'note', wrap: true }  — or controller.setColumnWrap(col, true)
controller.autoSizeRows({ measure, font: '13px sans-serif', lineHeight: wrapLineHeight(13) });
```

Wrapped columns paint multi-line (clipped to the row); non-wrap columns keep
the single-line ellipsis path with zero added cost.

### Custom editor registry

```tsx
const editors = new EditorRegistry();
editors.registerEditor('color-picker', ({ value, container, commit, cancel }) => {
  const input = document.createElement('input');
  input.type = 'color'; input.value = value || '#000000';
  input.onchange = () => commit(input.value);
  container.appendChild(input);
  return { focus: () => input.focus(), destroy: () => input.remove() };
}, { commitOnOutsideClick: true });

<LatticaGrid editors={editors} columns={[{ headerName: 'Color', field: 'c', editor: 'color-picker' }]} ... />
```

`commit(next)` joins the normal commit path (sanitize/validation/undo/
`cellcommit` source `'edit'`). Unregistered kinds silently fall back to the
text editor.

### Comments & tooltips

```ts
controller.setComment(row, col, '異常値検知');  // empty/blank deletes; marker paints top-right
controller.getComment(row, col); controller.hasComment(row, col);
```

```tsx
<LatticaGrid cellTooltip={(row, col) => anomaly(row, col) ? '判定基準範囲外' : null} ... />
// Hover tooltip (data-testid="lattica-tooltip", 500ms dwell). Comments win over cellTooltip.
// Theme token: commentMarkerColor (default '#d64545')
```

### Pinned summary (footer) rows

```tsx
<LatticaGrid
  summaryRows={[{ label: '合計', cells: { qty: 'sum', price: 'avg', memo: (vs) => `${vs.length}件` } }]}
  ...
/>
```

Aggregates (`sum|avg|min|max|count` or a custom fn) run over filtered visible
rows, re-compute on commit/filter/sort, render pinned below the body
(read-only, non-selectable), and apply the column number format. Theme tokens:
`summaryRowBackground`, `summaryRowTextColor`.

### `bar` cell type (pseudo-Gantt)

```ts
controller.setColumnType(6, 'bar');
// Cell value: '{"color":"#2563eb","label":"Build","ratio":0.6}' or a plain string label.
// Spans merged ranges; malformed JSON falls back to plain text. Drag-to-resize is future work.
```

## P0 batch — Handsontable-replacement critical APIs (2026-07-05)

Eight additive features completing the tayca P0 requirements
(`docs/PRODUCT_REQUIREMENTS_TAYCA.md`, PRs #70–#77).

### Cell-level meta layer (`cells()` equivalent)

```tsx
const pending = new Map<string, true>(); // "row:col" (physical)
<LatticaGrid
  cellMeta={(row, col) =>
    pending.has(`${row}:${col}`) ? { background: '#dbeafe', readOnly: true } : null}
  ...
/>
// pending.set('3:2', true); controller.refreshCellMeta();  // explicit repaint
```

`CellMeta = { background?, color?, fontWeight?, readOnly? }`, physical
coordinates, re-evaluated every scene build (no flicker by construction).
Style priority: read-only theme background < conditional format / search /
validation < **cellMeta** < selection/active chrome. `readOnly` ORs with
`setCellReadOnly` / `setColumnEditable`.

### Link / action cells

```ts
controller.setColumnType(0, 'link');   // or ColumnDef { type: 'link' }
```

```tsx
<LatticaGrid onCellAction={(e) => openDetail(e.value)} ... />
// e: { row, col, value, display }. Click or plain Enter on the active cell.
// Fires on read-only cells and view-only grids; hover shows a pointer cursor.
```

### Programmatic writes with options

```ts
controller.setCellText(r, c, '09:30', {
  source: 'my-revert',      // propagated to cellcommit (ignore your own writes)
  bypassReadOnly: true,     // write into read-only cells (linked time cells)
  undoable: false,          // keep out of the undo history
});
```

### Empty-cell placeholder hints

```ts
controller.setColumnPlaceholder(2, '0.00');   // or ColumnDef { placeholder: '00:00' }
// placeholderMode: 'editable' (default) | 'always'; theme token placeholderColor.
// Display-only: stored values, copy and edit text are unaffected.
```

### External edit commit / cancel

```ts
gridRef.current?.commitEditing();   // true if an edit was committed
gridRef.current?.cancelEditing();
import { commitAllEditing, cancelAllEditing } from '@ai-path/tb-react';
commitAllEditing();                 // sweep every mounted grid (open a modal safely)
```

### autoHeight (show all rows)

```tsx
<LatticaGrid autoHeight ... />
// Grid height = header + all rows + summary band. No internal vertical
// scroll (wheel passes through to the page). Horizontal scroll unchanged.
// Virtualization is bypassed — intended for report-sized row counts.
```

### Pixel-layout APIs

```ts
gridRef.current?.getRowClientRect(row);   // { left, top, width, height } | null
gridRef.current?.getRowClientRects();     // visible rows: [{ row, top, height }]
controller.getColumnWidth(col); controller.getColumnWidths();
<LatticaGrid onLayoutChange={() => repositionRail()} ... />  // rAF-debounced
```

### Imperative row insert / remove

```ts
controller.insertRow(3, { date: '2026-07-05' });  // array or field-record values
controller.removeRow(3);
// Single-command undo/redo; 'rowschange' event / onRowsChange prop;
// row-keyed cell meta (readOnly, comments) shifts with the rows.
```

## If you modify this repo

- Tests: **Vitest**, unit tests next to source (`foo.ts` → `foo.test.ts`).
  **100% coverage is mandatory** (lines/branches/functions/statements) — `vitest.config.ts`
  enforces it. Unreachable defensive branches use `/* v8 ignore next -- reason */`.
- Gate before commit: `npx vitest run --coverage` (100%), `npx eslint .` (0),
  `pnpm run typecheck`, `pnpm run build` — all clean.
- One feature = one PR; keep `docs/PROGRESS.md` updated.
- Demo app: `examples/playground` (Next.js). `pnpm --filter ./examples/playground dev`.
- E2E: Playwright specs in `e2e/` (`*.spec.ts`, kept out of the coverage glob).

---
> Source: [aipathjp/lattica](https://github.com/aipathjp/lattica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
