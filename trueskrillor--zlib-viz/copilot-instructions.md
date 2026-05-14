## zlib-viz

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A browser-based static web app that visualises the internal bit-level structure of zlib (RFC 1950), gzip (RFC 1952), and raw-DEFLATE (RFC 1951) streams. Three virtualized panes (bytes · structure · decoded output) cross-highlight each other as the user clicks or hovers.

Design spec: `docs/superpowers/specs/2026-04-21-zlib-viz-design.md`.
Original implementation plan: `docs/superpowers/plans/2026-04-21-zlib-viz.md`.

## Commands

```bash
npm run dev              # vite dev server
npm run typecheck        # tsc -b --noEmit   (run this after type-sensitive edits)
npm test                 # vitest run        (68 unit/integration/property tests)
npm test -- test/parser/bit-reader.test.ts    # single file
npm run test:watch       # vitest watch mode
npm run e2e              # Playwright smoke suite (launches its own dev server on :5173)
npm run build            # tsc -b && vite build → dist/

# Regenerate binary fixtures from source strings (after touching generate.ts).
# Remember to copy the gzip example to public/ as .bin (see "gzip gotcha" below):
npx tsx test/parser/fixtures/generate.ts
```

Tests use Vitest (not Jest). Property tests use fast-check with ~280 random inputs per format; parser output is compared byte-for-byte against Node's native `zlib`.

## Architecture — big picture

Four layers, each with one responsibility and a clear interface:

```
UI (React) ──selection/hover──▶ State (Zustand)
                                  │
                                  ▼
                        ParsedStream (immutable)
                                  ▲
                                  │ postMessage (structured clone)
                        Worker (parse-worker.ts)
                                  │
                                  ▼
                        Parser core (pure TS, no DOM)
```

### Single-buffer, bit-addressed model

Everything the UI highlights is expressed in **bit coordinates from `input[0]`**, via `BitRange = { start, end }`. That single convention is what makes cross-pane highlighting work without per-pane stream-walking logic. Every node the parser emits carries a `BitRange` (or enough bit offsets to build one). The parser is eager and single-pass; `ParsedStream` contains the fully decoded output as one `Uint8Array`.

### Parser core (`src/parser/`)

Pure TS, zero DOM imports — reusable from a CLI or tests without the UI. Key invariants:

- **LSB-first bit order.** RFC 1951 defines bits within a byte LSB-first. `BitReader` handles this; `buildHuffmanTable` bit-reverses each canonical code before indexing the flat lookup table.
- **`BitReader.peek(n)` zero-pads past EOF** (but `readBits` still throws). This is required because `decodeSymbol` always peeks `maxBits` ahead, and a valid DEFLATE stream can end with fewer than `maxBits` bits remaining after the end-of-block code; the LSB-first striped lookup resolves the short code correctly regardless of the padded high bits.
- **Error tiers** — fatal (stop parsing the current block; emit `partialParsed`), soft (e.g. ADLER32/CRC32 mismatch; keep going), unexpected (thrown; the worker harness converts to `{ type: 'error' }`).
- **The `dynamic-small.zlib` fixture must actually be dynamic.** Node's zlib picks fixed-Huffman for short inputs; `generate.ts` uses a diverse multi-sentence payload to guarantee BTYPE=2. If you change the input, run the generator and verify with `(bytes[2] >> 1) & 3 === 2`.

### Worker boundary (`src/worker/`)

One fresh worker per parse; input `Uint8Array.buffer` is transferred. The UI keeps its own copy via `new Uint8Array(bytes)` (which is a copy, not a view, when the arg is a TypedArray) and stashes it under `inputBytes` — see Task 18 wiring in `InputArea.onBytes`.

Progress events are currently emitted once per parse (not per block). The plan flags this as a known approximation; if streaming feedback becomes important, thread a callback through `parseDeflate`.

### State (`src/state/`)

One Zustand store (`useUiStore`). Three derived-state patterns to know:

- **`resolveSelection(selection, parsed)`** — the only place that maps a `Selection` to bit/output/backref ranges. Every pane subscribes to its output; panes never walk the stream themselves.
- **`findSelectionAtBit(parsed, bit)`** — reverse direction: clicking a bit resolves to the innermost structural node, drilling into HLIT/HDIST/HCLEN and code-length entries before falling back to symbol / block / wrapper / trailer.
- **`isExpanded(id, expansion)`** — per-row collapse state. Defaults vary by prefix: `block:*` defaults expanded, `symbols:*` defaults collapsed (a block with 10 000 symbols would otherwise drown the tree).

The `Selection` union has seven kinds: `none | wrapper | trailer | block | blockField | blockSection | symbol`. `blockField` paths can terminate at either a `BitRange` directly (e.g. `hlitRange`) or an object with a `.range` property — `asBitRange` in `resolve-selection.ts` handles both shapes. `blockSection` is synthesised: `huffman-tables` spans `[dynamicMeta.hlitRange.start, first-symbol-start)`, `symbols` spans `[first-symbol-start, block.range.end)`, and together with the 3-bit header they partition every block exactly.

### UI (`src/ui/`)

Three panes, each virtualised with `react-window`. Two non-obvious things:

- **Every `FixedSizeList` passes `style={{ overflowX: 'hidden' }}`.** react-window's default outer div is `overflow: auto`; row content is even a few pixels wider than the container (grid padding etc.) triggers a spurious horizontal scrollbar on top of the row.
- **Global `*, *::before, *::after { box-sizing: border-box }` is load-bearing.** react-window sets `width: 100%` on absolutely-positioned rows, and with the default `content-box` the row's padding would add on top of 100%, pushing content past the container and clipping the range label on the right.
- **`useMeasure` (ResizeObserver) feeds real pane heights into every list.** Don't hardcode heights.
- **Hex / bit-stream rows are responsive** via `rowBytesFor(width)` / `bitsPerRowFor(width)` — 8 bytes / 32 bits per row below 540 px of pane width, 16 / 64 above.
- **The tree is always rendered at full detail.** There is no depth selector; `buildTreeRows` emits every HLIT/HDIST/HCLEN field, the Huffman-tables region, and the symbols group for every Huffman block. The symbols group defaults collapsed (see `isExpanded` / `symbols:*` rule above) so large blocks don't drown the tree.

## Gotchas the codebase has already tripped on

- **Don't ship example fixtures with `.gz` extension.** Vite (and most static servers) set `Content-Encoding: gzip` for `.gz`, which makes the browser transparently decompress the response — `fetch().arrayBuffer()` then returns the decoded payload, not the gzip bytes. The in-app gzip example is served as `gzip-plain.bin`. Test fixtures under `test/parser/fixtures/` can keep `.gz` because vitest reads them from the filesystem.
- **When touching `parseDeflate`, don't remove the back-ref byte-by-byte copy loop.** LZ77 allows `distance < length` (the RLE trick); a `Uint8Array.set` with a `subarray` would read source that has already been overwritten.
- **`export type Symbol`** in `parser/types.ts` shadows the global `Symbol` in the type namespace. Known, benign; imports use `import type { Symbol as DeflateSymbol }` where ambiguity matters.
- **React-window `Row` callback deps matter.** When responsive layout changes `rowBytes`/`bitsPerRow`, the row function must be included in the `useCallback` deps or the list keeps rendering with the old value.

---
> Source: [TrueSkrillor/zlib-viz](https://github.com/TrueSkrillor/zlib-viz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
