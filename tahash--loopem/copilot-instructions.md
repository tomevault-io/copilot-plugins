## loopem

> Loopem is a draggable, snapping nav: items laid along a line or an arc, with the

# Loopem

Loopem is a draggable, snapping nav: items laid along a line or an arc, with the
centered one active. Plain TypeScript and the DOM — no UI framework, no runtime
dependencies. Everything composes as `createX(params)` returning an element or a
handle object.

## The core model

One idea carries the whole library, and most questions resolve by returning to it:

- **State is a single float.** `offset` is the continuous position in *item
  units*. The active index is `round(offset)`.
- **Layout is a pure function of distance from center.** `place(d)` takes
  `d = index - offset` and returns `{ x, y, rotate, scale, opacity, z }`.

Virtualization is then "render only `|d| <= radius`", infinite looping is "wrap
the index, never the geometry", and a custom layout is just another `place`. Keep
new features expressible in these terms; if something can't be, say so before
building it.

## Project structure

A single package. Two layers, and the dependency runs one way only:

- `src/core` — the engine: offset state, layouts, physics, virtualization
  windows, snap targets. **Pure. Must not touch the DOM or any browser global**,
  so it stays testable in node and reusable by every framework adapter.
- `src/dom` — the rendering and input layer (→ core): slot pool, transform
  writes, pointer/wheel/keyboard handling, structural CSS.

Alongside them, `src/layouts` holds ready-made layout presets (→ core). Nothing
in the library imports it — only consumers do, through `loopem/layouts` — which
is what keeps it out of core. It is pure, and node-tested like core. One file
per preset; a new one is that file plus a line in its `index.ts`.

`src/adapters` holds the framework adapters (→ dom), one file per framework and
one public subpath each. They are the only part of the tree allowed a peer
dependency, which is why they sit apart from the library proper.

`src/index.ts` is the curated public surface — the only thing consumers see.
A new adapter is a file here, an entry in `tsdown.config.ts`, and a line in the
exports map — never a new package.

## Commands

The repo uses pnpm. Build is [tsdown](https://tsdown.dev) (Rolldown-powered),
config in `tsdown.config.ts`.

- `pnpm build` — build to `dist/` (also typechecks via `--dts`).
- `pnpm check` — Biome format + lint check; `pnpm check:write` to autofix.
- `pnpm test` — Vitest (`tests/**/*.test.ts`); `pnpm test:watch` for watch mode.
- `pnpm typecheck:tests` — typecheck the `tests/` tree.

A green build is the type check; Biome enforces format/lint. CI runs format +
lint + build + typecheck-tests + test on every PR.

`docs/` and `examples/` are separate pnpm workspaces, each with its own lockfile
and its own `pnpm install`. Both resolve `loopem` to `src/` — docs through the
exports map's `development` condition, examples through a Vite alias — so
library edits hot-reload without a build. In `docs/`: `pnpm dev`, `pnpm build`
(Astro + Pagefind), `pnpm exec astro check` to typecheck.

## Formatting

Biome (`biome.json`): no semicolons, single quotes, 2-space indentation, 80-col
width. Run `pnpm format`. When something isn't specified here, follow the
conventions already used in the surrounding code.

Relative imports are extensionless (`from './layout'`, never `'./layout.js'`).

## Code style

- Group code into sections with banner comments, with tunable constants at the
  top of the file. The `–` rule lines run to the 80-col width: `// ` + 77
  en-dashes (`–`, U+2013). CSS banners are `/* ` + 74 en-dashes + ` */`.
  ```ts
  // –––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
  //  Section title
  // –––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
  ```
- Comment only when the code and names don't already explain it. Never restate
  the code, narrate a change, or reference a plan; comments must read standalone.
  Default to none.
- Use common, widely-understood names — never cute ones. The rAF driver is the
  *ticker*; "loop" always means the wrap-around behavior, never the frame loop.
- Any string-literal value used in more than one place becomes an UPPER_SNAKE
  `as const` object with a derived type (e.g. `LAYOUT`, `AXIS`). Never repeat raw
  enum strings.
- Build things as `createX(params)` factories returning an element or a handle
  (`{ element, ... }`) — no classes.
- Prefer `interface`. Avoid casts, and never use `as any`.
- Keep a helper local until a second use, then extract — don't duplicate, and
  don't abstract on the first use.
- Structural CSS only. The library ships what it needs to *work* (containment,
  absolute positioning, transform origins, `touch-action`) and nothing that is a
  *look* — no colors, fonts, radii, or sizes. Those belong to the consumer.

## Performance

This is a library whose entire value is that it stays smooth, so the frame path
is the hot path:

- One rAF ticker, and **no rAF at all while idle**.
- Per frame: writes only. Never read layout (`getBoundingClientRect`,
  `offsetWidth`, …) inside the ticker — measure on resize and cache.
- Cache the last transform string per slot and skip identical writes.
- `will-change: transform` only while moving; clear it on settle.
- Never create or destroy elements on a high-frequency event. Slots are pooled
  and recycled; a slot's DOM changes only when its item index changes.
- Physics runs on fixed-timestep substeps, so 60 Hz and 144 Hz feel identical.

## Testing

- Tests live in `tests/`, mirroring `src/` (no `src/` prefix in the path); never
  colocated with source. The Vitest glob is `tests/**/*.test.ts` only.
- Anything the public entry exports is imported `from 'loopem'`, so an
  export-map regression fails here rather than in a consumer's app. Internal
  core modules are imported relatively; routing their unit tests through a DOM
  handle would test less, not more.
- Black-box otherwise: exercise a module's private helpers through its own
  exported surface, so they stay free to refactor.
- `core` is pure and carries most of the suite. It runs in the node environment,
  so a core test reaching for a browser global means the logic is in the wrong
  layer. In `dom`, unit-test whatever a stub can assert — the ticker's frame
  gaps and its sleep behavior — and leave rendering and input to be verified by
  running an example under `examples/`.

## Boundaries

- Ask before: new dependencies, new public subpaths, or changes to the public API.
- Never: use `as any`; touch the DOM from `src/core`; read layout in the ticker;
  ship styling that is a look rather than a mechanism; write comments that
  narrate a change; repeat raw enum strings.

## Keeping this file useful

Add a rule only when it is non-obvious, recurring, and specific enough to act on.
Keep it high-signal, and remove rules that go stale.

---
> Source: [TahaSh/loopem](https://github.com/TahaSh/loopem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
