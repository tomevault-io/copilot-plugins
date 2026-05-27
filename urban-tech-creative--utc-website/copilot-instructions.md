## handover

> Session handover — current project state, immediate next steps, and open decisions. Updated at the end of each working session.


# Session Handover

Last updated: 2026-03-09 (session 3)

## What happened this session

Performance fix for the 3D cube (GPU black-box artifacts) plus favicon replacement.

### 1. Cube face baking — GPU fix

The cube was causing black-box rendering artifacts in the browser. Root cause: `mix-blend-mode` inside `transform-style: preserve-3d` with continuous RAF animation exhausts the GPU compositor. Fixed by replacing the live face component layouts with static JPEG images captured from Storybook.

**Baked faces (done):** XR (`public/faces/xr.jpg`), Work (`public/faces/work.jpg`), Hamster (`public/faces/hamster.jpg`)

**Unbaked faces (still live):** AI, Collaborators, Showcase — these are placeholder layouts with no blend modes, so they're not causing GPU issues.

**Pattern:** Each baked face component (e.g. `XR.tsx`) returns a plain `<img src="/faces/xr.jpg" ... />`. The original FaceGrid layout is preserved as a named const (`XRDesign`, `WorkDesign`, `HamsterDesign`) so redesign + re-bake is possible without losing the source.

**Capture script:** `scripts/capture-face.ts` — runs `npx tsx scripts/capture-face.ts <face>`. Navigates Playwright to the canonical `Cube/Faces` Storybook story, screenshots the face element at 300×300 `@2x DPR` → 600×600 JPEG q90. JPEG (not PNG) gives 70-84% smaller files vs PNG with no visible quality loss on these designs.

**Image format note:** Playwright `element.screenshot()` only supports `png` and `jpeg` — no WebP natively. JPEG at q90 is the pragmatic choice. This project is on Cloudflare/OpenNext where `next/image` WebP conversion requires paid Cloudflare Image Resizing; static JPEG from CDN is simpler and good enough.

**Preload hints:** `app/page.tsx` uses `ReactDOM.preload()` to hoist preload hints for all three baked face images so they fetch in parallel with other page resources.

**Storybook story size:** `Faces.stories.tsx` decorator bumped from `200px` to `300px` (matches cube max size, so Playwright captures at the correct source resolution).

### 2. Favicon replacement

Replaced the default Vercel triangle (`favicon.ico` — deleted) with the UTC cube logomark.

- `app/icon.png` — 256×256 RGBA PNG, transparent background. Browser tab favicon, auto-registered by Next.js App Router.
- `app/apple-icon.png` — 180×180 RGBA PNG, transparent background. iOS "Add to Home Screen" touch icon.

**Capture script:** `scripts/capture-favicon.ts` — single command captures both. Navigates Playwright to dedicated Storybook stories (`FaviconCapture`, `AppleFaviconCapture` in `Logo.stories.tsx`), uses `page.screenshot({ type: 'png', omitBackground: true })` for transparency.

**Transparency gotcha (important for future):** Three separate background layers needed clearing before `omitBackground: true` worked:
1. `html { background: var(--theme-black) }` — app's `globals.css` — cleared via `documentElement.style.background = 'transparent'`
2. `body.sb-main-fullscreen { background: white }` — Storybook class — cleared by stripping `document.body.className`
3. Storybook's layout wrapper — `<div style="position: fixed; inset: 0; background: rgb(255,255,255)">` as first child of `#storybook-root`, inline style — cleared via `storybookRoot.firstElementChild.style.background = 'transparent'`. This was the hard one: inline styles can only be overridden by other inline styles, so no stylesheet `!important` approach works.

**Favicon stories:** `FaviconCapture` (128×128 viewport @2x → 256×256) and `AppleFaviconCapture` (90×90 viewport @2x → 180×180) added to `Logo.stories.tsx`. Containers are viewport-filling (`100vw × 100vh`) so `page.screenshot()` captures exactly the viewport, avoiding clipping from the CSS layout box that the 3D-transformed cube visually overflows.

### 3. Cursor skill added

`.cursor/skills/bake-cube-face/SKILL.md` — LM skill for both face baking and favicon capture workflows. Triggered when user says "bake", "capture", "snapshot", or "freeze" a face.

---

## Current state of the design system

### Component hierarchy

| Layer | Components |
|-------|-----------|
| Utilities | Pressable, Icon |
| Atoms | Frame, Accent, Heading, UIGrid, Overlay |
| Molecules | Button (primary/secondary/tertiary), NavLink, Logo lockup |
| Organisms | SiteHeader, PrimaryNav, NavMenuPanel, Cube, SectionDetail |

### Key interaction patterns

- **Button** = drop-in, opinionated, self-contained (use for standard actions)
- **Pressable** = zero-opinion wrapper (use for custom compositions like Logo lockups, NavLinks)
- **Hover** = cyan bg (Button built-in) or `group-hover:` (Pressable compositions)
- **Active/press** = `translate-y-1` downward shift, 100ms (Button built-in)
- **Focus** = 4px magenta outline, 2px offset (Button built-in; add manually for Pressable compositions)

### Cube face status

| Face | Position | Status |
|------|----------|--------|
| XR | top | Baked → `public/faces/xr.jpg`, source in `XRDesign` const |
| Work | front | Baked → `public/faces/work.jpg`, source in `WorkDesign` const |
| Hamster | bottom | Baked → `public/faces/hamster.jpg`, source in `HamsterDesign` const |
| AI | back | Live placeholder — needs design |
| Collaborators | left | Live placeholder — needs design |
| Showcase | right | Live placeholder — needs design |

---

## Immediate next steps

1. **Design and bake the AI, Collaborators, Showcase faces** — use Storybook experiments → promote → `npx tsx scripts/capture-face.ts <face>`. These are the remaining undesigned faces.

2. **`cta` variant for Button** — discussed but not yet implemented. Level above `primary`. Visual direction: accent gradient (magenta-green). Naming agreed as `cta`.

3. **Inline link component** — needed eventually; `tertiary` Button is not appropriate for inline text links.

---

## Open decisions

- **`cta` variant visual treatment**: Accent gradient? How distinct from `primary`? Deferred.
- **Grid gap standardisation**: 4px tight / 8px separated proposed but not confirmed as a token.
- **Section icons in NavLinks**: CSS grid icons may be replaced with Phosphor icons (they break at small sizes).
- **UIGrid for page layouts**: Ready but not yet used in any actual page layout.

---

## Gotchas

- **Tailwind v4 `translate` vs `transform`**: `translate-y-*` uses CSS `translate` property, not `transform`. Transition must target `translate`.
- **Tailwind v4 `outline` with theme colors**: `outline-theme-cyan` doesn't work — use `outline-[var(--theme-cyan)]`.
- **Tailwind v4 `outline-none`**: Sets `outline: 2px solid transparent` which interferes with `focus-visible`. Use `outline-0` instead.
- **Storybook story imports**: Always `import type { Meta, StoryObj } from '@storybook/nextjs'` — NOT `@storybook/react`. ESLint rule `no-renderer-packages` enforces this.
- **Phosphor icon names**: Verify against `node_modules/@phosphor-icons/react/dist/index.d.ts`. Registry uses kebab-case.
- **Vercel/Cloudflare build vs local dev**: `next build` runs ESLint + TypeScript; `next dev` does not. Run `npm run build` before pushing if lint/type issues suspected.
- **Storybook + workerd hang**: `next.config.ts` gates `initOpenNextCloudflareForDev()` behind `!isStorybook` — never remove this gate or Storybook build hangs.
- **Playwright `waitUntil: 'networkidle'` times out on Storybook**: Storybook's HMR websocket keeps the network permanently active. Always use `waitUntil: 'load'` in capture scripts.
- **Playwright `element.screenshot()` clips 3D-transformed content**: CSS 3D transforms cause visual overflow beyond the element's layout box. For the favicon logomark, use `page.screenshot()` with a viewport-filling container instead.
- **Playwright `omitBackground: true` needs all background layers cleared**: See transparency gotcha above. Three layers, each requiring a different technique.

---

## Working style notes

The developer (Alex) prefers:
- Separate commits per concern (not monolithic refactors)
- Discussing design decisions before executing — use Plan/Ask mode for "what do you think?" conversations
- Semiotic naming over presentational (variant names describe *meaning*, not *appearance*)
- Practical over precious — prototype fast, extract patterns later
- Storybook as the primary design tool ("vibe-designing")
- Clear documentation that serves both humans and LLM agents
- Having a genuinely good time while building things
- Direct answers — if something is not worth doing, say so

---
> Source: [urban-tech-creative/utc-website](https://github.com/urban-tech-creative/utc-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
