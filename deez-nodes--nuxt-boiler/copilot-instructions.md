## nuxt-boiler

> > **TODO / investigate:** dev server renders at `http://localhost:3001/`, not `:3000`. Port 3000 was taken by another project running concurrently, so Nuxt fell back to 3001 — the app is fine, but confirm the expected dev URL and whether a fixed port should be pinned.

# TradeSlayer

> **TODO / investigate:** dev server renders at `http://localhost:3001/`, not `:3000`. Port 3000 was taken by another project running concurrently, so Nuxt fell back to 3001 — the app is fine, but confirm the expected dev URL and whether a fixed port should be pinned.

## Overview
Trading journal and strategy tracking application built with Nuxt 4 + Vue 3.

## Tech Stack
- **Framework:** Nuxt 4.3.1 (Vue 3.5)
- **UI Library:** Nuxt UI 4.5.1
- **Styling:** Tailwind CSS 4.2.1
- **Package Manager:** pnpm (not npm)
- **Language:** TypeScript

## Project Structure
```
app/
├── app.vue              # Root component (UApp > NuxtLayout > NuxtPage)
├── assets/css/main.css  # Global CSS imports (Nuxt UI + Tailwind)
├── layouts/default.vue  # Dashboard layout with sidebar nav
├── types/
│   └── strategy.ts      # Strategy card types (StrategyColor, CardMeta, etc.)
├── composables/
│   ├── useLLM.ts        # LLM chat interface (local LM Studio)
│   └── useStrategyColors.ts  # Shared color palette + hex-alpha utility
├── components/
│   ├── StrategyCard.vue         # Legacy simple checklist card (used on strategies page)
│   ├── TradingViewChart.vue     # TradingView widget wrapper
│   └── strategy/                # Strategy reference card system
│       ├── StrategyCardShell.vue     # Shared card shell (header, tabs, footer)
│       ├── StrategyCheckbox.vue      # Toggleable checklist item
│       ├── StrategySection.vue       # Collapsible accordion section
│       ├── StrategyPill.vue          # Inline colored badge
│       ├── StrategyDivider.vue       # Labeled separator
│       ├── StrategyRefTag.vue        # Cross-card reference link
│       ├── StrategyInfoBox.vue       # Colored info/alert box
│       ├── ImbalanceStrategyCard.vue  # Atomic: bid/ask imbalance
│       ├── AbsorptionExhaustionSpoofCard.vue  # Atomic: absorption/exhaustion/spoof
│       ├── CvdStrategyCard.vue        # Atomic: cumulative volume delta
│       ├── IcebergStrategyCard.vue    # Atomic: iceberg detection
│       ├── LargeLotsStrategyCard.vue  # Atomic: large lot prints
│       ├── LiquidityRegimeCard.vue    # Atomic: liquidity regime
│       ├── LiquidityTrackerProCard.vue  # Atomic: liquidity tracker
│       ├── SessionVolumeProfileCard.vue # Atomic: session volume profile
│       ├── StopRunStrategyCard.vue    # Atomic: stop run patterns
│       ├── SupportResistanceStrategyCard.vue  # Atomic: S/R levels
│       ├── VolumeBubblesStrategyCard.vue  # Atomic: volume bubbles
│       ├── MomentumStrategyCard.vue   # Composite: momentum continuation
│       ├── PullbackStrategyCard.vue   # Composite: pullback entries
│       └── ReversalStrategyCard.vue   # Composite: reversal plays
├── pages/
│   ├── index.vue        # Dashboard page
│   ├── journal.vue      # Journal page
│   ├── strategies.vue   # Strategies page (card grid)
│   └── strategies/
│       └── [id].vue     # Individual strategy card viewer
```

## Conventions
- Use `pnpm` for all package operations (never npm)
- Vue SFC files use `<script setup lang="ts">`
- CSS classes are extracted into `<style module>` blocks — no inline class strings in templates
- Use `$style.className` syntax in templates for CSS Module references
- Nuxt UI components (UDashboardGroup, UDashboardSidebar, etc.) are auto-imported

## Nuxt UI Dashboard Layout Pattern
The correct component hierarchy for dashboard pages:
```
UApp
└── NuxtLayout (wraps NuxtPage, NOT siblings)
    └── NuxtPage
```

Layout file (`layouts/default.vue`):
```
UDashboardGroup
├── UDashboardSidebar
│   ├── #header slot
│   └── UNavigationMenu
└── UDashboardPanel
    ├── #header slot → UDashboardNavbar
    └── #body slot → <slot /> (page content)
```
- `UDashboardPanel` uses named slots (`#header`, `#body`, `#footer`) — do NOT pass content via the default slot or the internal layout (overflow, padding, shrink) is lost
- `UDashboardPanel`'s `#body` slot provides `overflow-y-auto`, `p-4 sm:p-6`, and flex column layout — do not add `UPageBody` inside it (that component is for marketing/content pages)

## Nuxt UI Component Choices
- **UPageGrid** — responsive 3-col grid (`grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8`), use for uniform card grids
- **UPageColumns** — CSS masonry columns, use for variable-height content
- **UPageCard** — marketing/landing page cards with spotlight effects; use **UCard** for dashboard data cards
- **UPageBody** — adds `mt-8 pb-24 space-y-12`, designed for content/marketing pages, NOT dashboard panels
- Verify a component's valid `color`/`variant`/`size` against its generated theme file `.nuxt/ui/<component>.ts` (e.g. `.nuxt/ui/badge.ts`)

## Strategy Card Architecture
Converted from React JSX prototypes in `card_designs/` to native Vue 3 SFCs. (The `card_designs/` prototypes have since been moved out to a separate app — expect them absent from this repo; a working tree showing them deleted vs HEAD is intentional.)

### Intentional, not-yet-wired (do not "clean up" as dead code)
- `CardMeta.iconGradient`, `StrategyTab.color`, `StrategySection` `color`/`icon` props, and `hex()`/`hexAlpha()`/`palette` in `useStrategyColors` render nothing yet but are planned wiring — keep them.

### Design decisions
- **14 separate SFC files** (not a data-driven single component) for template clarity and debuggability
- **7 shared sub-components** in `components/strategy/` extract all repeated UI patterns
- **StrategyCardShell** wraps every card with header (icon, title, pills), tab bar, content slot, and footer
- **StrategyColor** type (`'long'|'short'|'warn'|'info'|'neutral'|'cyan'|'purple'|'orange'`) is the single source of truth for the color palette
- **useStrategyColors()** composable provides `hex()` and `hexAlpha()` — no raw hex strings in components
- **11 atomic cards**: single-indicator reference checklists (tabs: SETUP/READ/LONG/SHORT/RISK or similar)
- **3 composite cards** (Momentum, Pullback, Reversal): compose multiple indicators, have LOG tabs with reactive form state, cross-reference other cards via `StrategyRefTag`
- Cards are mounted at `/strategies/:id` via a dynamic route; the strategies index page shows a card grid linking to each

### Adding a new strategy card
1. Create `app/components/strategy/NewCard.vue` following the ImbalanceStrategyCard pattern
2. Add the card to the `allCards` array in `pages/strategies/[id].vue`
3. Add it to the grid in `pages/strategies.vue`

## TODO: Dashboard Chart Widget (`pages/index.vue`)
- Add the TradingView Advanced Chart widget to the dashboard page
- Default symbol should be MES (Micro E-mini S&P 500) or NQ (Nasdaq) — NOT AAPL or any single stock
- The chart should render inside the `UDashboardPanel` `#body` slot (same pattern as strategies page)
- Use `<style module>` for any chart container styling — no inline class strings

## Commands
- `pnpm dev` — Start dev server
- `pnpm build` — Production build
- `pnpm preview` — Preview production build
- Typecheck: `pnpm exec nuxi typecheck` (needs `vue-tsc`+`typescript` devDeps). First run fails `ERR_PNPM_IGNORED_BUILDS` — fix once with `pnpm approve-builds` (select esbuild, @parcel/watcher, vue-demi). Never bypass pnpm by calling binaries directly.

---
> Source: [deez-nodes/nuxt_boiler](https://github.com/deez-nodes/nuxt_boiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
