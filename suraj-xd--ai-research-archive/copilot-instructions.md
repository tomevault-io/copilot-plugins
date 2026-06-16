## ai-research-archive

> This is **AI Research Archive** — a Vite + React 19 + Tailwind v4 + TypeScript app, deployed on Vercel. It hosts a curated AI/ML curriculum, resource library, jobs feed, and community directory.

# CLAUDE.md — Project guide for Claude

This is **AI Research Archive** — a Vite + React 19 + Tailwind v4 + TypeScript app, deployed on Vercel. It hosts a curated AI/ML curriculum, resource library, jobs feed, and community directory.

## The design system is non-negotiable

The project ships under the **General Agents Design System** (canonical zip: `~/Downloads/General Agents Design System.zip`, mirrored visual reference in `/tmp/ga-ds/` during dev sessions). **Use it.** Don't introduce ad-hoc colors, shadows, fonts, or layout primitives.

### Where the system lives

| Area | Location |
|---|---|
| Tokens (colors, type, radii, shadows, spacing) | `src/index.css` — `:root { --ga-* }` block + `@theme inline` token bridge |
| Brand fonts (Overused Grotesk, IBM Plex Mono, Departure Mono) | `public/fonts/` + `@font-face` in `src/index.css` |
| Brand logos / wordmarks | `public/assets/brand/` + `public/logo.png` (the pink `research archive` mark, also the favicon) |
| Brand React primitives | `src/components/brand/` — `Logo`, `Sparkle`, `PhIcon`, `ZigDivider`, `Sidebar`, `TopBar` |
| App shell | `src/components/Layout.tsx` + `Header.tsx` + `Sidebar.tsx` (uses brand primitives) |
| shadcn/ui primitives, brand-skinned | `src/components/ui/` |
| Live design showcase | `/design` route — renders every primitive against the canonical preview HTMLs |

### Foundational rules

- **Light mode only.** Dark was intentionally dropped — keycap buttons, layered soft shadows, and warm off-white surfaces don't survive a token-only invert. Don't add a `.dark` block back without a deliberate redesign pass.
- **Page background is `#FBFBFB` (`--ga-bg`)**, never pure white. White (`--ga-surface`) is reserved for cards, inputs, popovers — surfaces that need to lift.
- **Cards** = white surface + 12px radius + soft layered shadow + **NO border**. Use the `.grid-card` class. Hover lift only fires when the card is `<a>` / `<button>` / `data-interactive="true"`. Static info cards stay flat.
- **Primary buttons** = the GA "keycap" — black gradient `linear(180°, #272727 → #414141)` with `inset 0 1px 1px rgba(255,255,255,0.48)` and `inset 0 -3px 0 #232323`. Use `<Button>` from `src/components/ui/button.tsx` (which wraps `.ga-btn-primary`). Never raw `<button class="bg-black">`.
- **Sentence case for headings.** ALL-CAPS only for monospace meta labels via `.ga-mono-label` (IBM Plex Mono, 11–12px, letter-spacing 0.08em).
- **No emoji. No exclamation marks.** The product is calm.
- **Iconography = Phosphor Icons** loaded via CDN (regular / fill / bold weights). Use `<PhIcon name="..." size={N} />` from `@/components/brand`. Never import from `lucide-react` for new code — the existing usages are residual.
- **Section breaks use `<ZigDivider label="…" />`** (the wavy hand-drawn line). Never a flat `<div className="h-px bg-border" />`.
- **Tabs / filter pills** use the `.ga-tab` class with `data-active={bool}`. Never bordered chrome.
- **Colored category badges are forbidden.** Tags/badges are neutral mono chips: `inline-flex px-2 py-0.5 rounded bg-secondary text-muted-foreground text-[10px] uppercase tracking-wider font-mono`. The only exceptions are third-party brand identity (YouTube red, GitHub black, Discord blurple) when shown as their own logo.

### Token quick reference

| Token | Value | Use for |
|---|---|---|
| `--ga-bg` | `#FBFBFB` | Page background |
| `--ga-surface` | `#FFFFFF` | Cards, inputs, popovers |
| `--ga-sidebar` | `#F8F8F8` | Sidebar bg |
| `--ga-hover` | `#EFEFEF` | Hover bg, active sidebar row |
| `--ga-chip` | `#F3F3F3` | Resting chip bg |
| `--ga-divider` | `#E9E9E9` | Hairline divider |
| `--ga-border` | `#E1E1E1` | Default border |
| `--ga-border-strong` | `#CFCFCF` | Popover ring, emphasized border |
| `--ga-fg1` | `#272727` | Primary text |
| `--ga-fg2` | `#929292` | Secondary text |
| `--ga-fg3` | `#ADADAD` | Tertiary / placeholder |
| `--ga-ink-900` | `#1D1D1D` | Button dark |
| `--ga-sparkle-from / -to` | `#A78BFA → #6366F1` | Ronika sparkle gradient (only saturated brand color — accent only) |
| `--ga-font-sans` | Overused Grotesk | UI body text |
| `--ga-font-mono` | IBM Plex Mono | Mono labels, numerics |
| `--ga-font-departure` | Departure Mono | Pixel-grid (brand wordmark, hero treatments) |
| `--ga-r-sm / md / lg / xl` | `6 / 8 / 12 / 16` px | Inputs / buttons / cards / modals |
| `--ga-shadow-sm / md / lg` | layered soft | Resting / elevated / overlay surfaces |

shadcn semantic vars (`--background`, `--foreground`, `--primary`, etc.) all reference the `--ga-*` tokens above. So `bg-background`, `text-muted-foreground`, etc. flow through the canonical palette automatically.

Legacy `--color-bg`, `--color-text`, `--color-text-muted` etc. utilities are aliased too — they exist for backward-compat in a few residual files and resolve to the canonical brand. Don't introduce new usages; prefer `text-foreground`, `text-muted-foreground`, `bg-card`, etc.

### When asked to build something new

Reach for these in this order:

1. **Existing brand primitive** (`Logo`, `Sparkle`, `ZigDivider`, `Sidebar`, `TopBar`, `PhIcon`)
2. **shadcn/ui primitive** (`Button`, `Card`, `Dialog`, `Input`, `Sheet`, `DropdownMenu`, `Tooltip`, `Badge`, `Separator`, `Label`)
3. **Canonical CSS class** (`.grid-card`, `.ga-btn`, `.ga-btn-primary`, `.ga-btn-secondary`, `.ga-btn-ghost`, `.ga-card`, `.ga-input-prompt`, `.ga-input-simple`, `.ga-chip`, `.ga-pill`, `.ga-pill-dark`, `.ga-tab`, `.ga-mono-label`, `.ga-mono-eyebrow`)
4. **Token via `var(--ga-*)`** if you need an inline style

Only fall back to ad-hoc Tailwind when none of those fit — and even then, never hardcoded hex colors. If you find yourself writing `text-blue-400 border-blue-400/30`, stop — that's the brutalist anti-pattern this project explicitly migrated away from.

### The landing page (`/`) is the one exception

`src/pages/LandingPage.tsx` runs a bespoke draggable card canvas with FK Grotesk Trial and glassmorphic chrome — it's intentional, not refactored to brand primitives because the visual language is its own. **But anything you ADD to the landing page must still draw from the design system tokens** (`--ga-font-mono`, `--ga-fg2`, `--ga-shadow-sm`, etc.) so additions feel cohesive with the chrome. The `SourcesBadge` in `LandingPage.tsx` is the reference example.

## Project structure

```
src/
  components/
    brand/         # Brand primitives (Logo, Sparkle, ZigDivider, Sidebar, TopBar, PhIcon)
    ui/            # shadcn primitives, brand-skinned
    landing/       # Draggable canvas (LandingPage internals)
    *.tsx          # Page-level composition components (Articles, Blogs, Roadmap, etc.)
  pages/           # Route components (mostly thin wrappers around `components/*`)
  data/            # Source-of-truth data files (curriculum, resources, papers, blogs, jobs, etc.)
  hooks/           # Reusable hooks (useGitHubStars, etc.)
  lib/utils.ts     # cn() helper from shadcn
public/
  fonts/           # Self-hosted Overused Grotesk, IBM Plex Mono, Departure Mono, FK Grotesk Trial
  assets/brand/    # GA wordmarks (general-agents-logo, ronika-sparkle, ronika-wordmark*)
  logo.png         # The pink "research archive" brand mark — also the favicon
```

## Commands

| Command | Use |
|---|---|
| `npm run dev` | Vite dev server on `http://localhost:5173` (HMR live) |
| `npm run build` | Production build (`tsc -b && vite build`) — **always run before claiming a task is done** |
| `npm run lint` | ESLint |
| `npm run preview` | Preview the production build |

## Conventions

- Path alias `@/` → `src/`
- TypeScript strict mode is **off** (`"strict": false`) — the codebase prioritizes velocity over exhaustive types. Don't add unnecessary type wrangling.
- React 19, no Next.js. Routing via `react-router-dom@7`.
- All routes are mounted in `src/main.tsx` under `<BrowserRouter>` with the `<Layout>` wrapper for everything except `/` (LandingPage) and `/design` (DesignSystemPage).

## Things to avoid

- Re-adding dark mode without a deliberate redesign of the keycap and shadows
- Adding `lucide-react` icons in new code (use Phosphor via `<PhIcon>`)
- Re-introducing colored category badges, hard 1px borders on cards, or flat horizontal section dividers
- Touching `LandingPage.tsx` chrome (the FK Grotesk hero, glass pills, draggable canvas) without an explicit brief — it's intentional
- Using `<button class="bg-black text-white">` instead of `<Button>` — the keycap shadows + insets are the brand
- Hardcoded hex colors anywhere except identifying third-party brand colors (YouTube red, etc.) inside source-attribution UI

---
> Source: [suraj-xd/ai-research-archive](https://github.com/suraj-xd/ai-research-archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
