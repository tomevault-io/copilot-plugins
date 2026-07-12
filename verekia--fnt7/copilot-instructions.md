## fnt7

> **FNT7** is a focused editor for stylized A-Z / a-z / 0-9 display fonts.

# CLAUDE.md

## Project

**FNT7** is a focused editor for stylized A-Z / a-z / 0-9 display fonts.
Sibling project to VCT7 — shares its corner-rounding bezier geometry,
SVG-as-project-file philosophy, and dark monospace aesthetic, but the data
model is glyph-centric instead of shape-centric.

Key concepts:

- **Named bezier presets** at the project level. A vertex (or whole shape)
  references one by name, so the same "sharp" preset can drive corners across
  every glyph. The first preset is the implicit default for shapes/vertices
  that don't override.
- **Lowercase falls back to uppercase**: any `a-z` glyph with zero shapes
  renders using the matching `A-Z` glyph. Applies in the canvas, grid view,
  text preview, and TTF/OTF/WOFF2 export.
- **SVG as project file**, round-tripped via `data-f7-*` attributes. The saved
  `.svg` opens in any browser as a glyph specimen; opening it in FNT7 restores
  the full editable project.
- **Font coordinates internally** (y up, baseline at y=0). The SVG render uses
  a flipping group transform so the file looks right in browsers.

Stack: Next.js 16 (pages router, static export) · React 19 (with React Compiler)
· TypeScript 6 · Zustand 5 · Tailwind v4 (PostCSS) · `bun test` · oxlint · oxfmt
· opentype.js (OTF emit).

## Package manager: Bun

Lockfile is `bun.lock` — do not generate `package-lock.json` or `yarn.lock`.

```sh
bun install
bun run dev          # http://localhost:3000
bun run all          # format:check + lint + typecheck + warden + test
```

Other scripts: `build` (produces `out/`), `start`, `typecheck`, `lint`,
`format`, `format:check`, `test`, `test:watch`.

## Layout

- `pages/_app.tsx` — Next.js app shell; imports `global.css`.
- `pages/index.tsx` — root page; lazy-loads `src/App` with `ssr: false`.
- `global.css` — Tailwind v4 entry + design tokens (same palette as VCT7).
- `src/App.tsx` — top-level editor; view-mode switch (grid / single / text).
- `src/types.ts` — `Glyph`, `Shape`, `ProjectSettings`, `BezierPreset`.
- `src/store.ts` — Zustand store; glyphs + settings + selection + history.
- `src/components/` — `Topbar`, `Toolbar`, `GridView`, `GlyphCanvas`,
  `TextPreview`, `ProjectPanel`, `GlyphInspector`, `Statusbar`.
- `src/lib/`:
  - `geometry.ts` — corner-rounding `pointsToPath` (verbatim from VCT7).
  - `glyph.ts` — glyph-level helpers (effective shapes with fallback,
    bezier-preset resolution).
  - `svg-io.ts` — project SVG read/write with `data-f7-*` metadata.
  - `font-export.ts` — OTF emission via opentype.js.
  - `file-system.ts` — File System API wrapper (Chromium).
  - `file-ops.ts` — high-level New/Open/Save/Export plumbing.
  - `snap.ts` — angle + grid snapping (verbatim from VCT7).
- Tests live next to their subject (`*.test.ts`), run via `bun test`. DOM
  globals are registered via `bun-test-setup.ts` (`@happy-dom/global-registrator`),
  preloaded through `bunfig.toml`.

## Coordinate system

- Internal: font-design space. x right, y **up**. Baseline at y=0,
  ascender ≈ +750, descender ≈ -200. Units per em ≈ 1000.
- SVG output: the same coordinate values are written into `d` attributes, but
  the whole document is wrapped in `<g transform="scale(1, -1)">` so it
  renders correctly (y-down) in browsers without changing the stored numbers.

## Bezier preset resolution

Resolution order at a vertex `i` of shape `s`:

1. If `s.pointBezierRefs[i]` is set → look up that preset name.
2. Else if `s.bezierRef` is set → look up that preset name.
3. Else → use `projectSettings.bezierPresets[0]` (the implicit default).

If a referenced preset name is missing (e.g. file edited externally), the
resolver falls back to the default preset rather than erroring.

---

## Behavioral guidelines

Same as VCT7's: think before coding, simplicity first, surgical changes,
goal-driven execution with `bun run all` as the standard final verification.

---
> Source: [verekia/fnt7](https://github.com/verekia/fnt7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
