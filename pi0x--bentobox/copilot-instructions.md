## bentobox

> Bentobox — an Apple-style bento grid generator. Vite + React 19 + TS + Tailwind v4.

# AGENTS.md

Bentobox — an Apple-style bento grid generator. Vite + React 19 + TS + Tailwind v4.
`pnpm dev` (the user usually has it running), `pnpm build`, `npx tsc --noEmit -p tsconfig.json`.

## Layout

```
src/bento/BentoCanvas.tsx  the board — one tree, used for BOTH preview and export
src/bento/render.tsx       Takumi export: PNG / WebP / SVG + clipboard copy
src/bento/markdown.ts      the board as a Markdown document, both ways (md4x)
src/bento/share.ts         the board as a URL fragment, both ways (base64url JSON)
src/bento/icons.ts         emoji vs. Iconify: fetch, cache, ink, search
src/bento/liquid-glass.ts  the refraction itself: raw RGBA in, raw RGBA out
src/bento/glass.tsx        that pass wired to a board — backdrop, rects, layer
src/bento/layout.ts        which cell each card lands in, and its rect in pixels
src/bento/{types,themes,presets}.ts   model, palettes, templates + canvas sizes
src/ui/{Preview,Sidebar,Field,AppearanceToggle}.tsx  app chrome; App.tsx owns all state
```

## Rules that keep this working

- **`BentoCanvas` is inline styles only.** It is mounted in the DOM *and* handed to
  Takumi's Rust layout engine; a stylesheet would be one more thing to keep in sync.
  Anything preview-only (selection ring, click target) must be gated on the
  `onSelect` prop so it cannot leak into an export.
- **App chrome is Tailwind, the board is not.** Don't mix.
- **Dark mode is chrome-only.** `app.css` redefines the `dark:` variant to follow
  `data-theme` on `<html>`; `useAppearance` mirrors the light/dark choice onto
  that attribute and the inline script in `index.html` resolves the same value
  before first paint, so a reload doesn't flash white — keep the two in sync. The
  OS preference only seeds the first run; after that the toggle is the only input.
  The board keeps its own `config.themeId`, so an export looks the same on any
  machine — never wire the appearance setting into `BentoCanvas`.
- **Takumi runs as WASM in the browser** — Vite's `browser` condition resolves
  `takumi-js` to it and pulls the 3.5 MB binary via a `?url` import. That import is
  why `vite.config.ts` has `optimizeDeps.exclude`; keep it.
- **`format` is a discriminated union** in Takumi's render options — pass a literal
  per branch, not a variable.
- **Takumi's `width`/`height` are output pixels, not CSS pixels.** `devicePixelRatio`
  scales the board's CSS pixels into that canvas, so a 2x export asks for a 2x
  canvas too (`render.tsx`). Passing the board's own size alongside `dpr: 2`
  silently renders the top-left quadrant at double scale and crops the rest.
- **An icon is an emoji or an Iconify name.** Emoji go through Takumi's emoji
  provider as glyphs — Geist (built into Takumi) has no dingbats, and a text
  presentation selector (`U+FE0E`) rasterizes as tofu, so add to `ICON_CHOICES`
  in `types.ts` only after rendering the glyph and looking at it. Anything
  shaped `prefix:name` is an Iconify SVG instead (`icons.ts`): fetched once into
  a module-level cache, `currentColor` swapped for the card's own ink, handed to
  both engines as an `<img>` data URI. `BentoCanvas` has to stay synchronous, so
  a cache miss falls back to the API's URL — `exportBento` awaits `preloadIcons`
  first so an export embeds bytes and not a link.
- **The Markdown tab round-trips through `markdown.ts`, and that has to stay
  exact.** `toMarkdown(fromMarkdown(doc))` is what lets the tab re-serialize on
  every keystroke without stealing the caret (`describes()` in `Sidebar.tsx`).
  Anything a card can hold must therefore be spellable: the escape hatches
  (`icon=""` for an emoji-led title, `icon="mdi:rocket"` for an icon with no
  glyph to lead with, `stat=` / `unit=`, an `&nbsp;` heading for a card with no
  headline, `### ` for a stat card's title) exist only for that.
  Add a field to `BentoCard` and you owe it a slot here — the smoke test below
  catches what you forget.
- **The URL fragment is the board's only persistence, so a link has to keep
  meaning what it meant.** `share.ts` writes the *whole* `BentoConfig` — never a
  diff against `defaultConfig`, which would halve the URL and silently repaint
  every link ever shared the day a template changes. Card ids are dropped (React
  keys, not content) and minted fresh on the way in; the payload carries a `v`
  and a decoder that doesn't recognise it returns `null`, meaning "leave the
  board alone", never "reset it". It is JSON rather than the Markdown of
  `markdown.ts` because `fromMarkdown` needs md4x's WASM: a board that can only
  arrive after an await flashes the default template first, and this one decodes
  in the `useState` initialiser.
- **Nothing arriving from the URL is trusted, and nothing about it touches the
  History API.** Every decoded value is clamped to `LIMITS`, matched against a
  closed set from `types.ts`, or dropped — those live next to the model precisely
  so the sidebar's fields and both parsers can't drift apart. `App.tsx` writes the
  hash with a debounced `location.replace`: a fragment-only replace stays on the
  page and leaves the history stack alone, where assigning `location.hash` would
  push an entry per keystroke. It does fire `hashchange`, so the listener that
  picks up a pasted board skips the app's own last write — decoding it would mint
  new ids and drop the selection mid-edit.
- **`DocEditor`'s two layers must lay out identically.** The textarea is
  transparent and sits over a `<pre>` of the same text in color, so anything
  affecting wrapping — font, size, line height, padding, `whitespace-pre-wrap`,
  `break-words` — belongs in the shared `layerClass` and nowhere else. The
  textarea's scrollbar is hidden rather than absent for the same reason: a
  visible one narrows its text box and skews every wrapped line. rangi
  guarantees the tokens concatenate back to the input, which is what keeps the
  caret over its glyph; assert that if you touch the grammar.
- **md4x is imported as `md4x/standalone`.** The package's root `types` condition
  points at a types-only file, so `import { parseAST } from "md4x"` does not
  typecheck; the subpath is also the exact build Vite serves the browser. It is
  WASM too — `initMarkdown()` before any `fromMarkdown()`.
- **Most themes are converted, not written.** The four authored palettes plus the
  two Geist ones are literal; everything after them in `themes.ts` is a rangi
  editor theme run through `fromRangi`, so `ThemeId` is `keyof typeof themes`
  and lives in `themes.ts` — `types.ts` imports the type straight back, which is
  free because `import type` is erased. The mapping is mostly obvious (`bg` is
  the surface, `fg` the ink, `cmnt` is already `muted`) but two things are not:
  an editor's canvas sits *behind* its cards here, so `surface` is `bg` and the
  canvas is pushed a step darker; and an editor's greys are tuned for reading
  one line at a time, so `legible()` walks anything under 7:1 body / 4.5:1 muted
  away from the background before it ships. Verify a converter change by
  rendering, not by reading hexes — `.tmp/audit` in the smoke recipe below
  measures every ink against the fill it lands on.

- **A filled card picks its own ink.** `inkOn()` answers with white until the
  fill gets too bright, which is what lets a pastel accent — half the imported
  themes have one — carry text at all; pass it every colour a gradient runs
  through, since it has to hold at both ends. `thinned()` then takes that ink
  down to muted at two different alphas, because white over a dark fill keeps
  its shape far longer than black over a light one. Accent cards are the one
  place left where muted text sits near 2:1: a saturated mid-tone fill has no
  room for it, and forcing the issue turns the eyebrow the same colour as the
  title.

- **Flex items need `min-w-0`.** The un-scaled 1200px board otherwise sets the
  preview column's min-content width and the page scrolls sideways.

## Liquid glass

Boards default to `material: "solid"`; `"glass"` is the other half of the switch.
A glass card paints no fill of its own: the pane is *in the backdrop image*, put
there by a post-process, and the card tree contributes only its tint, its rim
and its ink. Three consequences:

- **The refraction needs to know where the cards are, so `layout.ts` decides.**
  It runs CSS grid's own sparse row-major auto-placement, and `BentoCanvas` pins
  every card to the cell it picked (`gridColumn: "3 / span 2"`). Never go back to
  bare `span n` — that hands placement to two engines independently and the glass
  stops lining up with the card the moment they disagree.
- **Every length in `liquid-glass.ts` is a board length**, multiplied by the
  `scale` argument on the way in. The board is refracted at 0.5× for the live
  preview and 2× for an export; a constant left in render pixels looks like a
  scale-invariant tweak and is really a shadow that halves between the two.
- **The layer is one pass over all the panes, not one pass per pane.** Chaining
  passes feeds each card the previous card's output, so a neighbour's drop shadow
  lands on already-refracted glass. Each pixel picks its nearest pane and reads
  the pristine backdrop.

The layer reaches the two engines differently and both paths are load-bearing:
Takumi takes raw pixels (`<Bitmap>`), the DOM and the SVG writer take a PNG data
URL — `renderSvg` silently drops a raw-pixel image, which looks exactly like the
glass "not working" rather than like an error. And the DOM paints the absolutely
positioned layer *above* its in-flow siblings, so the grid is `position:
relative` to climb back over it; Takumi paints in document order and doesn't care.

Backdrops are stacked gradients in `themes.ts` and they are not decoration —
glass refracts what is behind it, so a flat fill comes back a flat fill and the
board reads as frosted plastic. Blooms with some hue variety are what make an
edge bend into something you can see.

## Verifying a board change

Don't trust the types — look at the pixels. Bundle a scratch entry with esbuild and
render through Takumi's native backend in Node (same layout engine as the browser
WASM one), then Read the PNG:

```bash
npx esbuild /tmp/smoke.tsx --bundle --platform=node --format=esm \
  --jsx=automatic --packages=external --outfile=.tmp/smoke.mjs && node .tmp/smoke.mjs
```

(must run from the project root so `takumi-js` resolves). This is how the tofu-glyph
bug was found.

For a Markdown change, render both sides and compare: a board and the same board
after `fromMarkdown(toMarkdown(config))` must produce byte-identical PNGs, for
every template.

For a URL change, decode every template's own hash back into a board: it must
render byte-identically, survive a hostile payload (absurd numbers, unknown
enums, a thousand cards) without throwing, and carry an emoji title — `btoa`
alone throws on one.

For a glass change, render the same board at 0.5×, 1× and 2× and put the three
side by side. They should differ only in resolution — matching rim widths,
matching shadow spread. That is the check a scale-dependent constant fails.

## Not built yet

Drag-and-drop placement (cards flow in order), saved boards beyond the URL,
custom fonts.

---
> Source: [pi0x/bentobox](https://github.com/pi0x/bentobox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
