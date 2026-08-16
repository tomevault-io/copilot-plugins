## kaenirr-web

> Static personal portfolio for Martin ("Kaenirr"), a 3D artist/programmer. Built with **Astro 6 + TypeScript**, served as static HTML/CSS, deployed to **GitHub Pages** on the custom domain **kaenirr.com**.

# Kaenirr_Web — Portfolio

Static personal portfolio for Martin ("Kaenirr"), a 3D artist/programmer. Built with **Astro 6 + TypeScript**, served as static HTML/CSS, deployed to **GitHub Pages** on the custom domain **kaenirr.com**.

## Why this stack

Astro outputs pure static HTML/CSS and ships **zero client JS by default**. The only scripts that reach the browser are small, hand-written, and opt-in per component: the theme toggle, the card tilt/glow, the skill-search filter, and the skill toast. The page shell and nav are authored once as components and reused across every page — no markup duplication. React/Next/Jekyll were rejected as heavier or worse to author for a low-interactivity site.

## Layout

```
src/
  layouts/Layout.astro       # <html> shell, <head>, FOUC-blocking theme script; mounts NavBar + SkillToast + tilt
  components/
    NavBar.astro             # left: name → "/" | center: nav links | right: ThemeToggle
    ThemeToggle.astro        # flips data-theme, persists to localStorage
    ProjectCard.astro        # project card: overlay link covers the card, skill tags clickable above it
    SkillTag.astro           # one skill pill; clickable (opens toast) iff its value matches a skill id
    SkillToast.astro         # global <dialog>; any [data-skill] click opens it — no navigation
  pages/
    index.astro              # home: round avatar + intro, vertically centered, then 4 square section buttons
    skills.astro             # search box + grid of skill cards
    experience.astro         # vertical alternating timeline, sorted newest-first
    projects.astro           # project grid (ProjectCard)
    contact.astro            # email / GitHub / LinkedIn cards (external links open in new tab)
  data/skills.ts             # single source of truth for skills (id, name, category, description)
  lib/duration.ts            # YYYY-MM month math; shared by the experience page and its client script
  nav.ts                     # nav items + site name
  scripts/tilt.ts            # cursor-tracking 3D tilt + underglow for any [data-tilt] element
  styles/theme.css           # color tokens (:root / [data-theme="dark"]) + shared .glow-card utilities
  assets/avatar.png          # source portrait; astro:assets derives the avatar, favicon, apple-touch-icon
public/                      # served as-is: CNAME (kaenirr.com)
tests/portfolio.spec.ts      # Playwright e2e
```

## Key patterns

- **Single source of truth.** Nav items live in `src/nav.ts`; skills live in `src/data/skills.ts`. The home buttons, nav bar, and skill tags all map over these — add an entry, not edits across files.
- **Skill click-through, globally, with no navigation.** `SkillToast.astro` is mounted once in the Layout and listens (delegated) for clicks on any `[data-skill]` element anywhere — experience, projects, or the skills page. It fills a `<dialog>` and `showModal()`s it. There are intentionally **no per-skill pages and no redirects**. `SkillTag` renders a known skill (id present in `skills.ts`) as a `<button data-skill>`; anything else becomes a plain, non-clickable pill.
- **Clickable card with clickable children.** A project card links to the project *and* contains clickable skill tags. Nesting interactive elements in an `<a>` is invalid HTML, so `ProjectCard` uses the overlay-link pattern: an absolutely-positioned `.card-link` covers the card (z-index 1), `.content` sits above it with `pointer-events: none`, and `.tags` re-enables `pointer-events` so the tag buttons stay clickable.
- **`.glow-card` is the shared visual primitive.** Defined in `theme.css`; used by home buttons, project cards, experience cards, skill cards. The tilt/glow transform is applied **only on `:hover`** — at rest the card has `transform: none` so text/SVG rasterize crisply (keeping them composited at rest caused blurry icons). `will-change: transform` is **not** in the CSS: a `:hover` rule promotes the layer in the same frame the animation starts, which is too late to help. `tilt.ts` sets it on `pointerenter` and clears it on `pointerleave` instead.
- **Tilt reads layout once per hover.** `tilt.ts` measures the card in *page* space on `pointerenter` (`rect.left + scrollX`) and compares against `event.pageX/pageY`, so `pointermove` never calls `getBoundingClientRect()` — no forced synchronous layout at pointer rate. Writes are coalesced into a single `requestAnimationFrame`; the cached measurement is refreshed only on `resize`.
- **Build-time dates go stale on a static site.** An ongoing role's duration is rendered at build time as a no-JS fallback and carries `data-ongoing-since`; a client script in `experience.astro` recomputes it from the same [src/lib/duration.ts](src/lib/duration.ts) so the page can't drift between deploys. Never compute a "now"-relative string only in frontmatter.

## Theming

- Colors are **CSS custom properties** in `theme.css`. `:root` holds light values; `[data-theme="dark"]` overrides them. Components reference `var(--…)` only — never hardcode a color.
- The active theme is stored in `localStorage` under `theme` (`"light"`/`"dark"`), defaulting to `prefers-color-scheme`.
- A **blocking inline script** in `Layout.astro`'s `<head>` sets `document.documentElement.dataset.theme` before first paint to prevent a flash of the wrong theme. `ThemeToggle` only flips the attribute and persists it.

## Routing & domain

Custom domain → site is served from root: `site: "https://kaenirr.com"` in `astro.config.mjs`, `base` left unset. `public/CNAME` holds the bare domain and is copied into `dist/` automatically. Do **not** set a `/<repo>/` base — that's only for `*.github.io` project pages.

## Deploy

`.github/workflows/deploy.yml` runs on push to `main`: **test → build → deploy**. It runs the Playwright suite, builds with `withastro/action`, and publishes via `actions/deploy-pages`. A red test run blocks the deploy. Repo Settings → Pages → Source = "GitHub Actions"; no `gh-pages` branch.

## Tests

Playwright e2e ([tests/portfolio.spec.ts](tests/portfolio.spec.ts)) is the safety net: nav resolves and brand link returns home; theme toggle flips, persists across reload and navigation; no FOUC; skills list + search filter; clicking a skill opens the toast without navigating (on the skills page, in experience, and in projects). The test webServer builds and previews the real output. `npm run check` runs Astro + TypeScript diagnostics (kept at 0 errors).

## Conventions

- TypeScript everywhere; strict mode on. Self-documenting names over comments; prefer small inlineable helpers.
- No hardcoded colors/spacing in components — go through the CSS tokens.
- Keep the shipped JS surface near zero; reach for a client script only when there's no static alternative.
- `set:html` is used only with hardcoded, developer-authored SVG strings (icons). It does **not** sanitize — never feed it user/remote data.

## Commands

```
npm run dev      # dev server at http://localhost:4321
npm run build    # static build → dist/
npm run preview  # serve the build locally
npm run check    # Astro + TypeScript diagnostics
npm test         # Playwright e2e (first run: npx playwright install chromium)
```

---
> Source: [Kaenirr/Kaenirr_Web](https://github.com/Kaenirr/Kaenirr_Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
