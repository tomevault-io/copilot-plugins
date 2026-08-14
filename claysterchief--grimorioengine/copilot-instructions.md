## grimorioengine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is this

Grimorio Engine is a pure CSS UI framework by ESC Labs with a PS1/CRT retro aesthetic. Zero external JS dependencies. Fonts are loaded from Google Fonts. The entire framework lives in one file: `packages/css/grimorio.css`.

`apps/showcase/index.html` is the **canonical component showcase** — it demonstrates every component and is the source of truth for all design patterns and usage. All JavaScript lives in `packages/core/grimorio.js` (the editable source; `js/grimorio.js` at repo root is a generated mirror for distribution — see "Two distribution channels").

**System requirements:** Node ≥18.0.0, npm ≥9.0.0

### Repository documentation map

Prose docs are in Spanish; this file is the developer entry point. When a task touches identity, tokens, or AI-generation, read the relevant doc rather than guessing:

| Doc | Read it when |
|---|---|
| `COMPONENTES.md` | You need the full utility/component catalog with examples — the AI manifest and de-facto component reference. |
| `ESC-LABS-PS1-FRAMEWORK.md` | The task touches brand identity, voice, typography, or the non-negotiable visual rules (source for "Identity constraints" below). |
| `USO-CON-IA.md` | Explaining or wiring up how an external AI generator consumes the framework (system-prompt + verification loop that pairs with `COMPONENTES.md`). |
| `PROYECTO.md` | You need the component/roadmap status board (what's done vs. partial). |
| `CHANGELOG.md` | Cutting a release or checking what changed between versions. |
| `README.md` / `packages/css/README.md` | Consumer-facing install/usage (GitHub-install + npm package page). |
| `llms.txt` | Root discovery index (llmstxt.org) linking the above. |

## Running / Developing

**Zero build step during development.** Edit `packages/css/grimorio.css` directly and see changes instantly with any static server. Choose one:

### Quick start (no npm):
```bash
python -m http.server 8080
# then open http://localhost:8080/apps/showcase/
```

### With npm:
```bash
npm install        # installs clean-css-cli devDependency (optional)
npm run serve      # npx serve . (any port) — then navigate to /apps/showcase/
```

### Before publishing:
```bash
npm run build      # minifies packages/css/grimorio.css -> packages/css/grimorio.min.css,
                    # mirrors both CSS files to css/ at repo root, and mirrors
                    # packages/core/grimorio.js -> js/grimorio.js (see "Two distribution channels")
npm run validate   # 9 invariants (see "Testing" below) — run before every commit
```

**Development workflow:** Open showcase in browser, edit CSS directly in `packages/css/grimorio.css`, refresh to see changes. No compilation or transpilation needed.

**Validation:** `.hintrc` configures HTMLHint validation (development only). The `no-inline-styles` rule is disabled by design — this policy is maintained manually in code review (see below).

## Common development tasks

**Add a new component:**
1. Create a new CSS section in `packages/css/grimorio.css` with the standard header pattern (see Architecture)
2. Use BEM naming in Spanish
3. Use CSS variables for all colors (never hardcode hex in components)
4. Add demo markup + example in `apps/showcase/index.html` under a `.separador` label
5. Test all three palettes (no class, `.cosmos`, `.crimson`)

**Update the showcase:**
- **Zero `style=""` anywhere** — `index.html` and all secondary pages are inline-style-free (verify with a grep for `style="` returning nothing). Solve every layout need with a utility or BEM class; if a value is missing, add the utility to `grimorio.css` first.
- Changes to `packages/css/grimorio.css` auto-reload in the browser; no build step needed
- Always add theme-switcher button to secondary pages (`data-tema` attribute + `#tema-flash` div required)

**Create a new secondary page (like login.html, contacto.html):**
- Copy boilerplate from existing secondary pages in `apps/showcase/html/`
- Update `menu-principal__item--active` on current nav item
- Secondary pages live at `apps/showcase/html/` (3 levels below repo root), so CSS and JS use a **`../../../` prefix** (`../../../packages/css/grimorio.css`, `../../../packages/core/grimorio.js`) — both link the editable source directly (zero-build dev), not the root mirrors. Images live at `apps/showcase/images/`, so they use `../images/`. Sibling pages use plain relative links (`contacto.html`).
- Paths to images use `../images/`

**Publish a release:**
1. Run `npm run build` — minifies `packages/css/grimorio.css` and mirrors both files into `css/` at repo root
2. Ensure `packages/css/grimorio.min.css` AND the root `css/grimorio.css` + `css/grimorio.min.css` mirror exist and are committed (the root mirror is what GitHub-install/CDN consumers get — see "Two distribution channels" below)
3. Tag the commit (e.g., `v2.0.1`)
4. Distribution via GitHub: `npm install github:ClaysterChief/GrimorioEngine#v2.0.1`

## Architecture

### Single-file CSS framework

`packages/css/grimorio.css` is organized into sections marked with this header pattern:

```css
/* ================================================
   SECTION NAME — Brief description
   ================================================ */
```

Sections follow a consistent ordering:

1. **Variables Globales** — all custom properties (palettes, transitions, semantic colors, border widths, font-size scale)
2. **Reset y Base** — html font-size, box-sizing, typography defaults
3. **Layout** — contenedor, grid, max-width helpers
4. **Utilities** (8 sections) — fondos, textos, flexbox, márgenes/paddings, display, alineación, tamaños de fuente, letter-spacing, posición/dimensiones
5. **Components** (~45 sections) — navegación, superficie, botones, formularios, tarjetas, imágenes, barras HP, carrusel, paginación, alertas, modal, tablas, ventana, consola, texto-máquina, stat-hud, VHS, animaciones, íconos CSS, acordeón, toast, spinners, tooltip, slider de rango
6. **Optimización** — accessibility + performance hints
7. **Responsive** — breakpoints (tablet + mobile)

**Critical CSS detail**: `html { font-size: 62.5%; }` makes `1rem = 10px` everywhere. All sizing uses rem.

### Theme system

Three palettes, applied as a class on `<body>` or any container:

| Palette | Class | Accent |
|---|---|---|
| Dual Signal (default) | *(none)*, or explicit `.dual` | `#b44fff` purple |
| Cosmos Blue | `.cosmos` | `#1a6fd8` blue |
| Crimson Signal | `.crimson` | `#e53935` red |

All component colors reference CSS custom properties (`--acento`, `--void-surface`, `--border`, etc.) so they automatically adapt when a palette class is applied. Theme switching triggers a CRT flash animation via `#tema-flash` + `.tema-flash--activo`.

**Nesting palettes**: `.cosmos`/`.crimson`/`.dual` all mirror the same structure (var overrides + background-color + background-image) so any of them can scope a nested element regardless of what palette an ancestor has. `.dual` exists specifically because Dual Signal has no class by default (it's just `:root`) — without it, a nested "no class" element would inherit an ancestor's `.cosmos`/`.crimson` variables instead of staying purple. Always use an explicit class (`.cosmos`, `.crimson`, or `.dual`) when showing one palette's look inside a container themed with another (e.g. theme-comparison cards) — never rely on "no class" to mean Dual Signal in that context.

### JavaScript (`packages/core/grimorio.js`)

All JS is vanilla ES6 in `packages/core/grimorio.js` (source), no modules, no bundler — everything is global scope. The showcase loads it directly via `<script defer src=".../packages/core/grimorio.js">`; installed consumers get the root mirror `js/grimorio.js` (via `exports["./js"]`). Initialized on `DOMContentLoaded`:

- `ESCCarrusel` — class, handles image/card carousels
- `ESCConsola` — class, interactive terminal with command history
- `iniciarFormPasos()` — multi-step form with sliding panels
- `iniciarTarjetaCredito()` — credit card 3D flip + live preview
- `iniciarCorreoInputs()` — email input with domain dropdown
- `iniciarSelectorTema()` — palette switcher with CRT flash (requires `<div id="tema-flash" class="tema-flash">` on the page)
- `iniciarTextoMaquina()` — measures natural width for typewriter CSS animation
- `iniciarArchivos()` — custom file input display
- `iniciarPaginacion()` — demo pagination
- `iniciarAcordeon()` — expandable accordion panels; supports `.acordeon--exclusivo` for single-open mode
- `iniciarRangos()` — range sliders with live value display; sets `--rango-pct` CSS variable
- `ESCToast` — singleton toast system; call `ESCToast.mostrar(msg, tipo, duracion)` where `tipo` is `info | ok | warning | error`

All functions return early if their target element is not found, so grimorio.js is safe to load on any page even if most components are absent. Theme choice is persisted in `localStorage['grimorio-tema']`.

### Naming conventions

BEM-simplified, **all class names in Spanish, kebab-case**:

```
.bloque
.bloque-modificador
.bloque__elemento
.bloque__elemento--modificador
```

Key prefixes: `btn`, `campo-`, `formulario-`, `tarjeta-`, `grupo-`, `imagen-`, `alerta-`, `tabla-`, `modal-`, `carrusel-`, `ventana-`, `consola-`.

### CSS variables (theme-aware)

```css
--void-deep       /* darkest background */
--void-primary    /* page background */
--void-surface    /* component surface */
--deep            /* elevated background */
--border          /* borders */
--acento          /* primary accent color */
--acento-light    /* highlight / glow */
--red-strike      /* action/alert accent */
--text-primary    /* body text */
--text-muted      /* labels, metadata */
```

Font variables (constant across palettes): `--font-display` (Orbitron 900), `--font-body` (VT323), `--font-mono` (Share Tech Mono) — this triad is brand identity, never swap for UI. One exception: `--font-code` (Cascadia Code PL, system-fallback stack, no @import) used only in `.codigo__cuerpo` for actual code legibility.

Additional token groups (the `:root` block in `grimorio.css` is the single source — see `--space-*`, `--z-*`, `--font-size-*`):

```css
--transition-fast / --transition-base / --transition-moderate / --transition-slow / --transition-card
--color-ok (#00c853) / --color-warning (#ffd600)
--color-dot-yellow / --color-dot-green
--border-thin (1px) / --border-accent (2px) / --border-thick (3px)
--space-0 / xs / sm / md / lg / xl / 2xl / 3xl / 4xl   (spacing — drives m-* / p-* / gap-*)
--z-base / z-nav / z-dropdown / z-sticky / z-modal / z-toast / z-flash   (z-index layers)
--font-size-xs / sm / base / md / lg / xl / 2xl / 3xl / 4xl   (+ --font-size-label/body aliases)
--header-height (5rem)   (sticky .cabecera height — drives global scroll-padding-top + .top-header)
```

**Token consistency is load-bearing** (the framework is meant to be AI-consumable — predictable scales prevent improvised values). All three spacing families (`m-*`, `p-*`, `gap-*`) share one **monotonic** `--space-*` scale, and every family covers the full range through `4xl` (side/axis padding `pt/pb/pl/pr/px/py-*` included — do not let it drift back to a truncated scale); `fs-*` is monotonic by name. Never introduce a token whose value breaks the ordering.

Do **not** tokenize: credit card internal colors (`#7a5f00`, `#0e0e0e`, `#f0f0f0`, etc.) — they simulate real card materials and must stay hardcoded.

### Utility class scale

Spacing scale (`--space-*`, used by `.m-*`, `.p-*` (incl. `t/b/l/r/x/y`), `.gap-*`, `.gap-x/y-*`). **Monotonic:**

| Token | Value |
|---|---|
| `0` | 0 |
| `xs` | 0.75rem (7.5px) |
| `sm` | 1rem (10px) |
| `md` | 1.5rem (15px) |
| `lg` | 2rem (20px) |
| `xl` | 3rem (30px) |
| `2xl` | 4rem (40px) |
| `3xl` | 5rem (50px) |
| `4xl` | 6rem (60px) |

Font-size utilities (monotonic): `.fs-xs` (1.2) · `.fs-sm` (1.3) · `.fs-base` (1.4) · `.fs-md` (1.5) · `.fs-lg` (1.6) · `.fs-xl` (1.8) · `.fs-2xl` (2.2) · `.fs-3xl` (3) · `.fs-4xl` (4.5rem). Aliases: `.fs-label` (=md), `.fs-body` (=xl). **Legibility floor 1.2rem/12px** — no framework text goes below 12px (VT323/Share Tech Mono are pixel fonts; sub-12px is hard to read). The low end was lifted from 10px; `lg`+ and body (`--font-size-body` = xl = 18px) are unchanged, so the type hierarchy is intact. Sole exception: the credit-card number (`1.05rem`, material simulation, like the card's hardcoded colors).

Letter-spacing: `.tracking-md` (0.1em) · `.tracking-wide` (0.08em) · `.tracking-wider` (0.18em)

Font family (family only): `.font-display` · `.font-body` · `.font-mono` (note: `.text-mono` also sets size + uppercase + tracking)

Position: `.pos-relative/absolute/fixed/sticky/static` · `.inset-0` · `.top-0/right-0/bottom-0/left-0` · sticky offset below header: `.top-header` (= `--header-height`) · `.top-sm/md/lg/xl` · anchor offset: `.scroll-mt-header/sm/md/lg/xl` (global `scroll-padding-top` on `html` already = `--header-height`)
Z-index: `.z-0/base/nav/dropdown/sticky/modal/toast` (map to `--z-*`)
Overflow: `.overflow-hidden/auto/scroll/visible` · `.overflow-x/y-auto` · `.overflow-x/y-hidden`
Opacity: `.opacity-0/25/50/60/75/100`
Width: `.w-full/half/fit/auto/screen` · fixed rem `.w-xs/sm/md/lg/xl` (12/20/24/28/32rem)
Height: `.h-full/auto/screen` · `.min-h-0/sm/md/lg/full/screen/vista` (vista = 100vh − 14rem)
Size (square, width=height + `flex-shrink:0`): `.size-xs/sm/md/lg/xl` (2/3/4/6/8rem) — swatches, avatars, square chips, dots
Min-width: `.min-w-0/xs/sm/md/lg/xl/full` (12/20/24/28/32rem)
Max-width: `.max-w-2xs/xs/sm/md/lg/xl/full` (32/40/48/56/68/80rem) · legacy aliases `.mw-xs/sm/md`
Aspect-ratio: `.aspect-square/video/4-3`
Line-height: `.leading-tight/snug/normal/relaxed` · text: `.truncate` · `.break-words` · `.text-nowrap` · `.no-subrayado`
Flex (added): `.jc-start/around/evenly` · `.ai-stretch/baseline`

Semantic helper classes (reusable patterns — prefer these over re-deriving inline): `.etiqueta` (mono eyebrow label, `--acento` modifier available) · `.regla` (subtle `<hr>` divider) · `.enlace` (accent link, no underline) · `.insignia` (generic inline badge/pill for use outside tables — variants `--acento/ok/advertencia/error`; inside tables use `.tabla-badge`) · `.migas` (breadcrumb `<nav>`; auto ◆ separators between items, `.migas__actual` marks current page)

Icon size modifiers: `.icono--sm` (1.2rem) · `.icono--md` (2rem) · `.icono--lg` (3rem) · `.icono--xl` (4.5rem)
Button modifier: `.btn--peligro` (recolors any button to `--red-strike`)

Animation utilities (expose existing `@keyframes` as classes; all honor `prefers-reduced-motion`): `.animate-glow/pulse/blink/glitch/scan/spin/ping/flicker/entrada` · control `.animate--once` · `.animate--pausa`
Transition utilities: `.transition` · `.transition-fast/base/moderate/slow` · `.transition-none`
Skeleton loader: `.skeleton` + `--titulo/--linea/--bloque/--avatar` (PS1 scanline shimmer)
Form validation: `.formulario-estandar--error/--ok` (also `.formulario-textarea--*`) + `.form-ayuda` / `.form-ayuda--error/--ok`
Accessibility: global `:focus-visible` ring in `--acento` (automatic, no class needed)

### HTML data attributes (JS hooks)

| Attribute | Used by | Effect |
|---|---|---|
| `data-tema="dual\|cosmos\|crimson"` | `iniciarSelectorTema()` | Switches palette + CRT flash |
| `data-paso-sig` | `iniciarFormPasos()` | Advance multi-step form |
| `data-paso-ant` | `iniciarFormPasos()` | Go back in multi-step form |
| `data-nombre-id="id"` | `iniciarArchivos()` | File input display target element ID |
| `id="tema-flash"` | `iniciarSelectorTema()` | Required CRT flash overlay (must exist on every page) |

## Testing

There is no unit-test runner (it would be a poor fit for a pure-CSS framework). Verification has two layers:

**1. `npm run validate` (`scripts/validate.mjs`, zero deps) — 9 invariants.** These encode the brand/architecture rules as assertions, so they can't silently rot: zero inline `style=` in `apps/`, `css/`+`js/` mirrors in sync with source, font-size floor ≥1.2rem, monotonic `--space-*`/`--font-size-*`, `border-radius` only `0`/`50%`, colors via `var()` (documented exceptions allowlisted by selector prefix), **`COMPONENTES.md`↔code contract** (every class the manifest teaches must exist in the CSS *or* be a JS behavior hook — this catches the recurring "manifest teaches a class that doesn't exist" bug), `#tema-flash` on every showcase page, and no orphan assets. Run it before every commit; exit code 1 fails CI.

**2. `apps/pruebas/` — five manual verification pages**, each isolating one risk class:

| Page | Covers |
|---|---|
| `matriz-paletas.html` | Every component in all 3 palettes side by side — catches the costliest silent bug: a component that ignores the theme |
| `limites.html` | Hostile content: unbreakable text, empty/huge lists, omitted optional elements, nested palettes |
| `responsive.html` | The showcase in iframes at 375/560/768/1024/1440px at once (real viewports, so media queries actually fire) |
| `accesibilidad.html` | Focus order, keyboard nav, contrast pairs, `prefers-reduced-motion`, ARIA |
| `js-api.html` | All 11 `grimorio.js` functions + all 7 Custom Elements, **loaded together on purpose** — that coexistence is itself under test |

**Custom Elements + `grimorio.js` coexistence:** the elements render the same markup `grimorio.js` auto-discovers, so both would bind and double-process clicks. Elements mark their rendered root with `data-esc-ce` and `grimorio.js` filters those out via `escFiltrarCE()`. **Any new auto-discovering init must use that filter**, and any new element that renders managed markup must set the marker.

## Adding new components

The showcase (`apps/showcase/index.html`) is the **source of truth** for all components, patterns, and best practices.

1. Add a new section in `packages/css/grimorio.css` with the standard header comment (see Architecture).
2. Follow BEM naming in Spanish.
3. Use CSS variables for all colors — never hardcode hex values in component rules.
4. Demonstrate the component in `apps/showcase/index.html` under a `.separador` label.
5. Test it across all three palettes to ensure theme switching works.

**Critical rule:** `apps/showcase/index.html` must not use `style=""` attributes. The showcase is the framework's own demo — every layout need must be solved with a utility class or BEM modifier. If a utility class for a needed value doesn't exist, add it to the Utilities sections of `packages/css/grimorio.css` first.

**New components must be visible and working in the showcase before being considered complete.** This is both the demo and the test suite.

## Secondary pages

`apps/showcase/html/login.html`, `apps/showcase/html/contacto.html`, and `apps/showcase/html/servicios.html` are complete. All three share the same shell:

- `<link rel="stylesheet" href="../../../packages/css/grimorio.css" />`
- `<script defer src="../../../packages/core/grimorio.js"></script>`
- Same nav with `menu-principal__item--active` on the current page's item
- `<div id="tema-flash" class="tema-flash"></div>` — required for theme switcher
- Same footer with `../images/` paths (images live at `apps/showcase/images/`)

Nav link paths within `apps/showcase/html/`: siblings use relative paths (`contacto.html`, `servicios.html`, `login.html`); Inicio links to `../index.html`.

## Identity constraints (from ESC-LABS-PS1-FRAMEWORK.md)

- The brand name is **ESC Labs** — never expanded in copy, no periods between letters.
- Approved taglines: *Systems that evolve.* / *Build. Evolve. Compute.* / *From science to system.* / *Tecnología con alma propia.*
- The `◆` diamond symbol is a recurring visual motif — used in separators, buttons, and decorative elements.
- No border-radius on structural elements — sharp corners are intentional to the PS1 aesthetic.
- No emojis in any brand material or UI copy.
- **`--red-strike` / `#ff3d6e`** is an action-only accent: CTAs, blinking cursors, HP bars, active indicators. Never use it as body text color or background fill.
- **Orbitron minimum size**: 18px. Below that it loses character — use Share Tech Mono for labels instead.
- **Scanlines** are required on heroes and main surface backgrounds (repeating-linear-gradient creating 4px stripes at ~20% opacity). They're already baked into `.superficie` and related component rules — preserve them when overriding backgrounds.

## Monorepo structure

The repo uses npm workspaces (`"workspaces": ["packages/*"]`). 

**Production-ready:**

| Package | Path | Purpose |
|---|---|---|
| `@grimorio/css` | `packages/css/` | ✅ Active — the complete CSS framework (`grimorio.css` + minified `grimorio.min.css`) |

**AI-consumable artifacts** (keep in sync with `grimorio.css` whenever tokens/components change — these are what an AI generator reads to produce on-brand UI):

| File | Purpose |
|---|---|
| `COMPONENTES.md` | The **AI manifest** — identity rules, palettes, token scales, full utility + component catalog with minimal examples, delivery checklist. Paste into a generator's system prompt. |
| `llms.txt` | Discoverable root index (llmstxt.org convention) linking the manifest + key files. |
| `packages/core/tokens.json` | Design tokens extracted from `:root` (single source: colors per palette, spacing, font-size, z-index, transitions, borders). `tokens.js` re-exports them as an ES module. |

**In progress / future frameworks (private, not published yet):**

| Package | Path | Purpose |
|---|---|---|
| `@grimorio/core` | `packages/core/` | Design tokens — `tokens.json` (source) + `tokens.js` (re-exports the JSON) — **and** `grimorio.js`, the vanilla-JS runtime source (mirrored to root `js/` by the build) |
| `@grimorio/elements` | `packages/elements/` | ✅ Web Components v0.3.0 — **Fase C complete**, 7 Custom Elements (light DOM, global `@grimorio/css`, no shadow DOM, self-registering ESM). Simple: `<esc-acordeon>`+`<esc-acordeon-item>`, `<esc-spinner>`, `<esc-tooltip>`, `<esc-toast>` (`ESCToast.mostrar()` static). Complex (reimplement `grimorio.js` logic in-element): `<esc-carrusel>`+`<esc-carrusel-item>`, `<esc-form-pasos>`+`<esc-paso>`, `<esc-tarjeta-credito>`. One file per element in `src/`; `index.js` registers all; `demo.html` verifies. Elements render standard Grimorio markup — accordion titles omit `◆` (CSS `::before` adds it). |
| `@grimorio/vue` | `packages/vue/` | 🚧 Vue 3 wrappers v0.1.0 — thin render-function components (no SFC → zero build) over `@grimorio/elements`: props→attributes, default slot→content, `useToast()` composable, default-export install plugin. `index.js` + `demo.html` (import map, CDN Vue) + `README.md`. Caveat: container elements rewrite their light DOM, so slot content is static after mount (use `:key` to remount for reactive lists). |
| `@grimorio/angular` | `packages/angular/` | Angular component wrappers (future) |
| `@grimorio/react-native` | `packages/react-native/` | React Native components (future) |

`packages/core/grimorio.js` is the JS **source** (edited directly, linked by the showcase). `js/grimorio.js` at repo root is a **generated mirror** (produced by `npm run build`, committed) — this parallels the CSS channels exactly: `packages/css/grimorio.css` (source) → `css/grimorio.css` (mirror). **Never hand-edit `js/grimorio.js` at root.** Installed consumers reach the mirror via `exports["./js"]`.

### Two distribution channels (don't confuse them)

1. **`@grimorio/css` (npm registry, scoped package)** — lives in `packages/css/` (unmoved). This is the **live-edited source** — the showcase links here directly (`packages/css/grimorio.css`), preserving the zero-build-step dev workflow. Files: `packages/css/grimorio.css` + `packages/css/grimorio.min.css`.
2. **`grimorio-engine` (root package — GitHub install / CDN like jsDelivr)** — **generated mirrors** `css/` and `js/` at repo root. Produced by `npm run build` (minify `packages/css/grimorio.css` → `.min.css`, copy both CSS into `css/`, and copy `packages/core/grimorio.js` → `js/grimorio.js`) and **committed to git**, so GitHub-install/CDN consumers don't need to build. **Never hand-edit `css/` or `js/` at the root** — both are regenerated by `npm run build` (and `npm run validate` fails if they drift from source).

**What consumers actually get** (root `package.json` `files` + `exports`) — verified end-to-end via `npm pack` → install → browser:

| Specifier | Resolves to | Notes |
|---|---|---|
| `grimorio-engine` · `/css` | `css/grimorio.css` | mirror |
| `grimorio-engine/css/min` | `css/grimorio.min.css` | mirror |
| `grimorio-engine/js` | `js/grimorio.js` | mirror, vanilla runtime |
| `grimorio-engine/elements` | `packages/elements/index.js` | ships `src/` too (7 elements) |
| `grimorio-engine/vue` | `packages/vue/index.js` | ships alongside elements |

`elements` and `vue` ship **as sources under `packages/`** (not mirrored to root) because they're ESM with relative imports — no build needed. This is why `packages/vue/index.js` imports `./../elements/index.js` **relatively** rather than the bare `@grimorio/elements`: the bare specifier only resolves inside the workspace, so it would break for every external consumer. **Keep that import relative.** Anything added to `packages/elements/` or `packages/vue/` must also be listed in the root `files` array or it silently won't ship.

The showcase lives at `apps/showcase/`: `index.html` is the component demo, `html/` has the secondary pages, `images/` has all demo assets. To open locally: `apps/showcase/index.html` in browser, or `npm run serve` and navigate to `/apps/showcase/`.

Stub packages carry `"private": true` to prevent accidental publishing before they have real code.

Planned dependency graph when complete:
```
@grimorio/core (tokens)
      ↓
@grimorio/css (styles)
      ↓
@grimorio/elements (Web Components)
      ↓              ↓              ↓
@grimorio/vue  @grimorio/angular  @grimorio/react-native
```

## Using as a package

Install directly from GitHub (no npm publish required):

```bash
npm install github:ClaysterChief/GrimorioEngine
```

Then import CSS globally in any bundler project:

```js
import 'grimorio-engine'      // -> css/grimorio.css
import 'grimorio-engine/js'   // -> js/grimorio.js (optional, for interactive components)
```

For **React**, manage theme with state rather than `data-tema` + `iniciarSelectorTema()` (that function targets DOM directly and doesn't fit React's lifecycle). Apply the palette class on `document.body` via `useEffect` and persist in `localStorage['grimorio-tema']`. Interactive components (carousel, multi-step form, credit card) should be ported to React hooks (`useEffect` / `useRef`) — the CSS doesn't change, only the JS logic moves into the component lifecycle.

---
> Source: [ClaysterChief/GrimorioEngine](https://github.com/ClaysterChief/GrimorioEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
