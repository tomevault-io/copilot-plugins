## rezuma

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev            # Vite dev server
pnpm build          # vue-tsc --noEmit && vite build  (type-check is part of the build)
pnpm preview        # serve the production build
pnpm preview:cf     # serve it the way Cloudflare will (wrangler dev)
pnpm deploy         # build + wrangler deploy
pnpm fonts          # re-download the vendored TTFs into src/assets/fonts/
pnpm verify:pdf     # generate PDFs and assert the text extracts back out
```

pnpm only (`packageManager` is pinned; `pnpm-workspace.yaml` allowlists build scripts wrangler needs). Node >= 20.19.

There is no unit-test runner and no linter. The only automated check besides `vue-tsc` is `pnpm verify:pdf`, which is the project's real test suite — it runs nine cases across three languages, two themes and four layouts, and exits non-zero on any miss. To narrow it, edit the `cases` array in `scripts/verify-pdf.mjs`; there is no filter flag. Run it after any change to `src/services/pdf/**`, `src/config/themes.ts`, `src/config/layouts.ts`, the vendored fonts, or the jsPDF stub aliases in `vite.config.ts`.

To see the app rendered without a browser extension, there is a Playwright Chromium at `~/AppData/Local/ms-playwright/chromium-*/chrome-win64/chrome.exe`: start `pnpm dev --port 5199`, then `chrome.exe --headless=new --disable-gpu --window-size=1700,1400 --virtual-time-budget=8000 --screenshot=out.png http://localhost:5199/app`. Language and layout are session state with no URL parameter, so reaching a specific combination means temporarily editing `DEFAULT_LANG` / `DEFAULT_LAYOUT`.

## What this is

Rezuma: a zero-backend, single-page Vue 3 app that edits a resume, runs live ATS checks on it, and exports a real vector PDF. No server, no account, no network at runtime. Read `PRODUCT.md` before changing anything user-facing — it defines the two registers (`/` is brand, `/app` is product), the anti-references (commercial resume-builder patterns), and the accessibility floor (WCAG 2.2 AA, AAA contrast where achievable).

## Architecture

`@/` resolves to `src/`. Files are grouped by role, not by feature.

**Three layers of state, deliberately separate:**

1. `useResume` (`src/composables/useResume.ts`) — the document. Module-level singleton, not a per-component store. It holds *every* language's document simultaneously, keyed off `LOCALES`; the active locale from `src/i18n` decides which one is on screen. Persists to `localStorage['my-resume:v1']` as `{ en, fr, ar }`. Undo/redo is per-language JSON snapshots with a 600 ms idle coalescing window, so a run of keystrokes becomes one step. The editor binds to the reactive object *directly* — never replace or re-clone the graph while typing, or the caret jumps mid-word.
2. `useTheme` (`src/composables/useTheme.ts`) — the *document* look **and** its layout. Nine styles in `src/config/themes.ts` and four layouts in `src/config/layouts.ts`, both pushed onto `<html>` as CSS custom properties. Document tokens are always paper: no dark mode, ever. This composable is also where the document's font family becomes direction-aware — in Arabic it asks `familyForTheme` for the face the exporter will embed, rather than re-deriving it, so the preview and the PDF cannot drift.
3. `useAppearance` (`src/composables/useAppearance.ts`) — the *interface* light/dark. `data-theme` on `<html>` always holds a resolved `'light' | 'dark'` (never `'system'`); the preference lives in `localStorage['my-resume:appearance']`. An inline script in `index.html` resolves it before first paint — **it and `useAppearance.ts` must agree on the storage key and on writing a resolved value**, or dark-mode readers get a white flash.

`src/styles/variables.css` documents the three token systems (chrome, app surface, document) and why they don't share a scheme. Don't blur the app/document line — it's design principle #3 in PRODUCT.md.

**PDF export (`src/services/pdf/`)** typesets the document with jsPDF; it does *not* rasterise the preview.

- `fonts.ts` — Source Serif 4 / Source Sans 3, plus Noto Naskh Arabic / Noto Sans Arabic, as raw TTF imported via `?url` and fetched at export time (kept out of the initial bundle). `times`/`helvetica` are jsPDF base-14 and carry no Arabic, which is why `familyForTheme(theme, lang)` takes the language. `EMBEDDED[].files` is deliberately partial — the Arabic faces have no italic, and `styleFor()` folds the gap rather than registering the same 180 KB file twice. Both live here so the toolbar can call them without pulling jsPDF into the initial chunk.
- `layout.ts` — two-pass typesetter. Everything is measured into `Group`s of `Item`s first, then `paginate()` places them. Groups carry `atomic` / `keepWithNext` / `orphans` / `widows`, which is what keeps a heading with its first entry and a job title with its dates. `paginate()` takes an array of `Flow`s: each restarts at page 1 with its own cursor, sharing the page sequence, which is how the sidebar layout gets a second column. **Flow order is the extracted text order** — the flow with the name and work history goes first regardless of which side it prints on.
- `exportPdf.ts` — turns a resume + theme + layout into those groups. Written entirely in *logical* coordinates: `anchorX`/`ruleSpan` take an offset from the leading margin and a width, and are the only place that knows which physical edge that is. Don't reintroduce a bare `MARGIN_X` into a call site. `fetcher` and `output: 'blob'` exist so `scripts/verify-pdf.mjs` can drive it from Node.

The exporter is imported dynamically (`await import('@/services/pdf/exportPdf')`) at the moment the user clicks Download, and `/app` itself is a lazy route. Keep it that way. `vite.config.ts` aliases `html2canvas`, `canvg` and `dompurify` to `src/services/pdf/unused-jspdf-deps.ts` — jsPDF's optional deps back `.html()` and SVG rasterisation, which this app never calls; unaliased they add ~105 kB gzip. `verify:pdf` is what proves the exporter still works without them.

**Vendored fonts.** jsPDF can only embed raw TrueType and `@fontsource` ships woff2, so `scripts/fetch-fonts.mjs` pulls static TTFs into `src/assets/fonts/`. They are committed so builds are hermetic; the OFL travels with them. Arabic has two traps the script guards against: the CSS API returns the *latin* slice unless `subset=arabic` is passed (24 KB, empty Arabic cmap, embeds fine and draws nothing), and jsPDF shapes by substituting Arabic Presentation Forms-B rather than running OpenType, so a face without U+FE70–FEFF in its cmap exports blanks. The script reads the cmap and refuses either.

**Arabic / RTL.** `setLocale` writes both `lang` and `dir` onto `<html>`; every stylesheet is in logical properties, so nothing else needs a direction branch — keep it that way. Three things about jsPDF are worth knowing before touching the exporter: it shapes and bidi-reorders automatically via `preProcessText`/`postProcessText` events (nothing to call); `getTextWidth` runs the same shaping pass so measurement is accurate, but `splitTextToSize` does *not*, which is why `wrap()` has a separate measured greedy path for RTL; and extracted text comes back as presentation forms, so any comparison against it needs `normalize('NFKC')`. Bidi moves whole runs and mirrors brackets, so a mixed Arabic/Latin line does not extract contiguously — `verify-pdf.mjs` compares RTL cases token by token for that reason, splitting on script boundaries as well as on punctuation.

**Layouts (`src/config/layouts.ts`)** — four arrangements, orthogonal to the themes. One file drives both renderers: `rail`/`gutter` become CSS custom properties for the preview's grid and column geometry for the exporter, and `aside` decides which sections move to a second column in both. The preview's two `.resume-column` wrappers are `display: contents` for every layout except `sidebar`, so they don't perturb anything else; sections are written into them in the order the exporter emits them, which is what keeps the DOM order, the copy-paste order and the PDF text stream identical. If you add a layout, add it to the `cases` in `verify-pdf.mjs` — `expectedOrder()` derives the expectation from the config, so a new layout is covered as soon as it is listed.

**Landing effects (`src/components/effects/`)** — a set of Vue Bits-style animations ported natively: no `motion-v`, `gsap`, `ogl` or `three`. The aurora is one fragment shader on a fullscreen triangle in raw WebGL2; the dot grid and click sparks are canvas 2D. Two reasons not to add the libraries: `/` is the *eager* route (only `/app` is lazy), so anything here lands in the entry chunk, and Vue Bits is MIT + Commons Clause while this repo is MIT. Only the aurora is big enough to matter and it is a `defineAsyncComponent`.

Three invariants for anything added to that directory:
- **Reduced motion is the *finished* state.** Read `useReducedMotion`; render the settled appearance and add animation on top. Never gate content behind an animation — a CSS rule that starts at `opacity: 0` must live inside `@media (prefers-reduced-motion: no-preference)`, which is how `effects.css` is organised.
- **Don't change what is announced.** Split/scramble/count keep an intact copy in a `.visually-hidden` sibling with the animated copy `aria-hidden`. `DecryptedText`'s `plain` prop is the deliberate exception, used in the extracted-text panel where a hidden duplicate would double every line on copy.
- **Loops must be visibility-bound.** Use `useVisibleRaf` (IntersectionObserver + `visibilitychange`) rather than a bare `requestAnimationFrame`.

The aurora composites differently per theme — `mix-blend-mode: screen` on dark, `multiply` on light with the shader emitting an opaque mostly-white frame. Plain alpha compositing of the dark output over a near-white ground turns the hero grey; that is what the `uLight` branch exists to avoid.

**ATS checks (`src/services/ats.ts`)** — `analyseResume(resume, layout) → AtsReport`, recomputed as a `computed` on every keystroke, so keep it pure and cheap. It takes the layout only so `doc-format` can tell the truth: a two-column layout is reported as a `warn`, because the linear text stream `verify:pdf` proves is not what a geometry-based parser reads. Every `Check` carries a `section`, which `EditorView` uses to focus the offending field via `ResumeEditor`'s exposed `reveal()`. The file's header comment draws the distinction that matters: the *file* being parseable is proven by `verify:pdf`; everything in `ats.ts` is about whether the *content* survives matching. The regexes (`METRIC_RE`, `FIRST_PERSON`, `WEAK_OPENERS`) cover all three languages and are carefully tuned against false positives like "Vue 3" or the French "2ème" — read the comments before loosening one. `METRIC_RE` carries the `u` flag for the Arabic alternatives' `p{…}` classes, and those alternatives use explicit lookarounds rather than ``, which is ASCII-defined and therefore matches in exactly the wrong places around Arabic. The Arabic `FIRST_PERSON` entries deliberately under-reach: person is marked morphologically in Arabic, and matching the affixes would fire on correct third-person bullets.

**i18n.** vue-i18n, Composition API only (`legacy: false`; `vite.config.ts` defines the `__VUE_I18N_*` flags to drop the legacy layer). `src/i18n/locales/en.ts` is the reference schema and `fr.ts`/`ar.ts` are typed `MessageSchema` against it, so a missing translation is a type error. `locales/index.ts` is the registry: adding a locale there (code, label, `dir`) is most of what a new language needs, since `useResume` keys its documents and history off `LOCALES`. The active locale is the single source of truth for language across the app — the resume store, the date formatters and `<html dir>` all read it. `INTL_LOCALES` maps Arabic to `ar-u-nu-latn` on purpose: CLDR would otherwise give Arabic-Indic digits, which no date parser recognises. The landing page is translated too and has its own language picker. ATS check messages are English in every locale by design.

`src/utils/normalise.ts` is the trust boundary: everything entering from `localStorage` or an imported JSON file goes through `normalise()` + `looksLikeResume()`, which check shape rather than presence.

## Constraints to preserve

- **Nothing leaves the browser.** No analytics, no fonts from a CDN, no network calls at runtime. The README makes this promise publicly; breaking it is a credibility failure, not just a defect.
- **No destructive action without undo.** "Reset to sample" goes through `replace()` and is undoable; that's the pattern, not confirmation dialogs.
- Reading order in the PDF is the parser's only structure — `paginate()` must keep drawing items in reading order, and a layout that adds a column must emit the flow carrying the name and work history first.
- The preview and the export must agree. Every layout, font and spacing decision exists twice — once in CSS, once in `exportPdf.ts` — and both sides read the same config (`themes.ts`, `layouts.ts`, `familyForTheme`). When you change one, change the other, and prefer deriving over duplicating.
- Deploy targets are static-only. `wrangler.jsonc` uses `not_found_handling: "single-page-application"`; `vercel.json` mirrors it. Workers rejects a `public/_redirects` SPA rule, which is why there isn't one — Pages/Netlify would need it added.

---
> Source: [boussadjra/rezuma](https://github.com/boussadjra/rezuma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
