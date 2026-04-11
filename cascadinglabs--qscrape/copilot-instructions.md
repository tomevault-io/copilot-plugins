## qscrape

> QScrape is a web scraper evaluation suite. It hosts fictional test sites across three difficulty levels to benchmark scraper capabilities. Built with [Astro](https://astro.build/). Made by [Cascading Labs](https://cascadinglabs.com), used for [Yosoi](https://github.com/CascadingLabs/Yosoi).

# QScrape — Project Documentation

## Overview
QScrape is a web scraper evaluation suite. It hosts fictional test sites across three difficulty levels to benchmark scraper capabilities. Built with [Astro](https://astro.build/). Made by [Cascading Labs](https://cascadinglabs.com), used for [Yosoi](https://github.com/CascadingLabs/Yosoi).

## Levels

| Level | Status | Description |
|-------|--------|-------------|
| L1 | Live | Standard HTML/CSS/JS. Static Astro build. No frameworks, no anti-bot measures. |
| L2 | Live | Modern web frameworks (React, Vue, Svelte, Solid). All content client-side only (`client:only`). Scrapers must execute JS. Each page section is an independent island assigned to a randomly shuffled framework — all 4 frameworks appear on every page load. |
| L3 | Live | Anti-bot sites. Astro islands (React + Vue + Svelte + Solid). No recaptchas or unsolvable challenge puzzles. |

## Index Pages

| Route | File | Description |
|-------|------|-------------|
| `/` | `src/pages/index.astro` | Root index: project overview, levels list, all sites, resources |
| `/l1/` | `src/pages/l1/index.astro` | L1 index: level explanation, list of L1 sites with descriptions |
| `/l2/` | `src/pages/l2/index.astro` | L2 index: level explanation and list of L2 sites |

### Shared CSS
Both index pages use `public/qscrape.css` — the dark-theme stylesheet for QScrape meta/navigation pages. This is separate from `public/global.css`, which is the legacy ASP.NET stylesheet used by L1 site pages.

## L1 Sites

| Route | Site Name | AGENTS.md |
|-------|-----------|-----------|
| `/l1/news/` | Mountainhome Herald (news portal) | `src/pages/l1/news/AGENTS.md` |
| `/l1/scoretap/` | ScoreTap (esports scores) | `src/pages/l1/scoretap/AGENTS.md` |
| `/l1/taxes/` | Eldoria Registry of Deeds (tax records) | `src/pages/l1/taxes/AGENTS.md` |
| `/l1/eshop/` | VaultMart (e-commerce catalogue) | `src/pages/l1/eshop/AGENTS.md` |

See each site's `AGENTS.md` for full page structure, data schemas, URL encoding schemes, and CSS architecture.

## L2 Sites

All L2 page sections are independent `client:only` islands. HTML source is empty shell — scrapers must execute JavaScript to see any content.

### L2 architecture (important for scraper authors)

**Islands, not SPAs** — each page is divided into named slots (e.g. `product-grid`, `category-nav`). Every slot embeds four copies of its component (one per framework), all `hidden` initially. An inline synchronous script runs a Fisher-Yates shuffle and reveals exactly one framework per slot. All four framework runtimes still hydrate; only the revealed copy is visible.

**Framework distribution** — the shuffler assigns a unique framework to each slot via permutation, so all 4 frameworks (React, Vue, Svelte, Solid) appear on every page load. The active framework is recorded in `data-active-fw` on each slot element.

**Async loading gate** — every island calls `fakeGet(null)` on mount, which resolves after a simulated 300–550 ms delay. Until resolved the island renders a plain "Loading…" div. Scrapers must wait for all visible islands to settle before extracting data. The fake API is at `src/data/api.ts`.

**Client-side URL routing** — navigation within a site uses `history.pushState` (query-param based). `popstate` listeners keep back/forward buttons functional. Taxes sub-pages (`/l2/taxes/how-to/` and `/l2/taxes/recording-fees/`) are separate Astro pages with their own single island slot.

| Site | URL state keys |
|------|----------------|
| news | `?id=…` (article detail), `?cat=…` (category filter) |
| eshop | `?sku=…` (product detail), `?view=cart\|checkout` |
| scoretap | `?match=…`, `?team=…`, `?event=…`, `?game=…` |
| taxes | `?file=…` (deed viewer), `?lastFirm=…&first=…&index=…` (search) |

**Game filter (scoretap)** — a single `activeGame` state controls filtering. The header tabs and per-section filter tabs update state via custom events, keeping all slots in sync regardless of which framework handles each slot.

### Shared data layer

| File | Used by |
|------|---------|
| `src/data/news/articles.ts` | All 4 news implementations + L1 |
| `src/data/eshop/products.ts` | All 4 eshop implementations + L1 |
| `src/data/scoretap/data.ts` | All 4 scoretap implementations |
| `src/data/taxes/deeds.ts` | All 4 taxes implementations |

### CSS design tokens

| File | Site |
|------|------|
| `src/styles/l2/news.css` | Mountainhome Herald (all frameworks) |
| `src/styles/l2/eshop.css` | VaultMart (all frameworks) |
| `src/styles/l2/scoretap.css` | ScoreTap (all frameworks) |
| `src/styles/l2/taxes.css` | Eldoria Registry (all frameworks) |

### Component locations

Components are organised **site-first**: `src/components/l2/{site}/{framework}/`. Each directory contains multiple `*.tsx` / `*.vue` / `*.svelte` files, one per slot.

| Site | React | Vue | Svelte | Solid |
|------|-------|-----|--------|-------|
| News | `src/components/l2/news/react/` | `…/vue/` | `…/svelte/` | `…/solid/` |
| Eshop | `src/components/l2/eshop/react/` | `…/vue/` | `…/svelte/` | `…/solid/` |
| ScoreTap | `src/components/l2/scoretap/react/` | `…/vue/` | `…/svelte/` | `…/solid/` |
| Taxes | `src/components/l2/taxes/react/` | `…/vue/` | `…/svelte/` | `…/solid/` |

**News slots**: `NewsBreakingTicker`, `NewsArticleFeed`, `NewsCategorySidebar`, `NewsStaffSpotlight`, `NewsWeatherWidget` (breaking news + geomantic conditions)

**Eshop slots**: `EshopCategoryNav`, `EshopFeaturedBanner`, `EshopPriceBadges`, `EshopProductGrid` (full cart/checkout flow)

**ScoreTap slots**: `ScoretapGameFilter`, `ScoretapLiveScores`, `ScoretapUpcoming`, `ScoretapRankings` (home page); `ScoretapEventsList` (`/events/`); `ScoretapTeamsList` (`/teams/`)

**Taxes slots**: `TaxesSearchForm`, `TaxesResultsGrid`, `TaxesIndexSidebar`, `TaxesRecentRecords` (+ sub-page single-slot islands: `TaxesHowTo`, `TaxesRecordingFees`)

## L3 Sites

L3 uses Astro Islands architecture. Each page composes four independent `client:only` islands — one per framework — each owning a different slice of the page data. Scrapers must hydrate all four runtimes and wait for four staggered async gates to reconstruct a complete record. Navigation between routes is full page loads (not SPA routing).

### L3 anti-bot techniques (per framework)

| Framework | Technique | How to defeat |
|-----------|-----------|---------------|
| React | CSS-assembled text — words split into `<span>` elements, DOM order shuffled, CSS `flex order` restores visual order | Must read `order` style on each span, sort by it, then join |
| Vue | Pseudo-element content — key data set as `data-x` attribute, rendered via `::before { content: attr(data-x) }` | Must use `getComputedStyle(el, '::before').content` |
| Svelte | Decoy overlay — real value at z-index 1, visually-identical fake value at z-index 2 with `color: transparent; position: absolute` | Must resolve z-index stacking to identify the real value |
| Solid | Canvas-rendered text — numbers and IDs drawn via `ctx.fillText()` to `<canvas>`; nothing in DOM | Must render canvas and OCR or use accessibility label (`aria-label`) |

### L3 page-level defenses

Every L3 page includes `L3Guard.astro` (`src/components/l3/L3Guard.astro`), which adds three page-level anti-bot measures. Additional per-site defenses (`ShadowShield.astro`, `PressHoldGate.astro`) are applied to specific sites.

| Defence | Mechanism | How to defeat | Sites |
|---------|-----------|---------------|-------|
| DevTools detection → 404 | Intercepts F12, Ctrl+Shift+I/J/C, Cmd+Opt+I/J/C keyboard shortcuts and replaces the entire document with a static 404 page. Also monitors `outerWidth − innerWidth` delta (polled every 500 ms + resize listener) to detect docked DevTools. | Disable JavaScript keyboard event listeners, or use a headless browser that doesn't trigger key events. For docked detection, use undocked DevTools or set matching outer/inner dimensions. | All L3 |
| Right-click disabled | `contextmenu` event is `preventDefault()`-ed, blocking the "Inspect Element" menu item. | Override the listener via `removeEventListener` or inject before the guard script runs. | All L3 |
| Honeypot buttons | A fixed `<div>` with `opacity: 0.001` contains four fake interactive elements (`Submit`, `Load All`, `Export`, `Reset`) with realistic class names and `data-action` attributes. Invisible to users, visible in the DOM to scrapers. | Filter elements by computed opacity or skip elements inside `aria-hidden="true"` containers. | All L3 |
| Shadow DOM encapsulation | After all islands hydrate (~2.5 s), `ShadowShield.astro` moves `<main>` content into a closed `attachShadow({ mode: 'closed' })` custom element. `document.querySelector()` from the main document returns nothing. | Monkey-patch `Element.prototype.attachShadow` before the page script runs to capture the shadow root reference, or use `element.getRootNode()` from inside the shadow tree. | Eshop |
| Press-and-hold gate | `PressHoldGate.astro` renders a full-screen overlay requiring a 5-second continuous press-and-hold before `<main>` is revealed. Progress is shown via an SVG ring animation. Completion is stored in `sessionStorage` so repeat visits bypass the gate. | Set `sessionStorage.setItem('hn3-verified', '1')` before navigation, or dispatch synthetic `mousedown`/`mouseup` events with a 5 s gap. | News (articles) |
| No source comments | All L3 component source files are stripped of `//`, `/* */`, and `<!-- -->` comments — no hints about anti-bot technique or implementation details. | N/A (code analysis required to understand obfuscation). | All L3 |

### L3Guard environment variable

L3Guard is controlled by the `PUBLIC_L3_GUARD` environment variable. It defaults to enabled (guard active) when the variable is unset or any value other than `"false"`.

| Variable | Default | Effect |
|----------|---------|--------|
| `PUBLIC_L3_GUARD` | (unset = enabled) | Set to `false` to disable L3Guard (honeypot buttons, DevTools detection, right-click blocking), ShadowShield (eshop shadow DOM), and PressHoldGate (news hold-to-verify) at build time. Useful for local development and testing. |

To disable locally, add to `.env`:

```
PUBLIC_L3_GUARD=false
```

The check runs at Astro build time in `L3Guard.astro` frontmatter — when disabled, the component renders nothing (no honeypot div, no inline script).

### L3 async loading gates (`fakeGetMs`)

Every L3 island calls `fakeGetMs(data, baseMs, jitterMs)` from `src/data/api.ts` on mount. Delays are staggered so all four islands finish at different times (worst case ~1050ms total):

| Framework | Base delay | Jitter | Resolves after |
|-----------|-----------|--------|---------------|
| Solid | 300ms | ±200ms | 300–500ms |
| React | 400ms | ±250ms | 400–650ms |
| Vue | 600ms | ±250ms | 600–850ms |
| Svelte | 800ms | ±250ms | 800–1050ms |

Until the promise resolves, each island renders a plain `Loading…` div. Scrapers must wait for all four islands to settle.

### L3 sites

| Route | Site Name |
|-------|-----------|
| `/l3/news/` | Mountainhome Herald (news portal) |
| `/l3/eshop/` | VaultMart (e-commerce catalogue) |
| `/l3/scoretap/` | ScoreTap (esports scores) |
| `/l3/taxes/` | Eldoria Registry of Deeds (tax records) |

### L3 island data split

Each page's data is split across four islands. A complete record requires all four.

**News — Mountainhome Herald**

| Route | React | Vue | Svelte | Solid |
|-------|-------|-----|--------|-------|
| `/l3/news/` | Breaking ticker + top 3 headlines | Article feed (title + excerpt + category) | Category sidebar | Staff spotlight panel |
| `/l3/news/articles/` | Category filter tabs | Article card grid | Pagination controls | Tag cloud |
| `/l3/news/article/[id]/` | Headline + byline + date | Article body text | Article image + caption + credit | Author bio + related links |

**Eshop — VaultMart**

| Route | React | Vue | Svelte | Solid |
|-------|-------|-----|--------|-------|
| `/l3/eshop/` | Category nav + sort controls | Product cards (image + name) | Price + stock badges (joined via `data-sku`) | Featured/promoted banner |
| `/l3/eshop/product/[sku]/` | Product image | Product name + category | Price + availability + add-to-cart | Rating + review count + related products |

**ScoreTap**

| Route | React | Vue | Svelte | Solid |
|-------|-------|-----|--------|-------|
| `/l3/scoretap/` | Game filter header tabs | Events list (teams, time, game) | Live scores ticker | Standings/leaderboard |
| `/l3/scoretap/teams/` | Team search input | Team cards (name, game, abbreviation) | Win/loss record per team | Top performers stats |

**Taxes — Eldoria Registry of Deeds**

| Route | React | Vue | Svelte | Solid |
|-------|-------|-----|--------|-------|
| `/l3/taxes/` | Search form (submits GET `?q=`) | Results list (reads `?q=` on mount) | Pagination + result count | Recent records sidebar |
| `/l3/taxes/viewer/[id]/` | Document header (parcel ID, type, date) | Owner name + address + assessed value | PDF download link + doc type badge | Related documents + recording fees |

### L3 CSS design tokens

| File | Site | Token prefix |
|------|------|-------------|
| `src/styles/l3/news.css` | Mountainhome Herald | `--hn3-` |
| `src/styles/l3/eshop.css` | VaultMart | `--vm3-` |
| `src/styles/l3/scoretap.css` | ScoreTap | `--st3-` |
| `src/styles/l3/taxes.css` | Eldoria Registry | `--er3-` |

L3 CSS token files do **not** contain `body {}` or `* {}` global resets — those live in each shell's `<style>` block, because all four islands share the same page.

### L3 component locations

| Framework | News | Eshop | ScoreTap | Taxes |
|-----------|------|-------|----------|-------|
| React | `src/components/l3/react/news/` | `…/eshop/` | `…/scoretap/` | `…/taxes/` |
| Vue | `src/components/l3/vue/news/` | `…/eshop/` | `…/scoretap/` | `…/taxes/` |
| Svelte | `src/components/l3/svelte/news/` | `…/eshop/` | `…/scoretap/` | `…/taxes/` |
| Solid | `src/components/l3/solid/news/` | `…/eshop/` | `…/scoretap/` | `…/taxes/` |

### L3 dynamic routes

Dynamic routes use `getStaticPaths()` — resolved at build time with props passed to each island:

| Route | Param | Data source |
|-------|-------|-------------|
| `/l3/news/article/[id]/` | `id` | `src/data/news/articles.ts` |
| `/l3/eshop/product/[sku]/` | `sku` | `src/data/eshop/products.ts` |
| `/l3/taxes/viewer/[id]/` | `id` | `src/data/taxes/deeds.ts` |

### L3 implementation notes

- Solid directive is `client:only="solid-js"` (not `"solid"`)
- React and Solid both process `.tsx` — `astro.config.mjs` uses `include` scoping to prevent JSX transform conflict: `react({ include: ['**/l2/react/**', '**/l3/react/**'] })` and `solid({ include: ['**/l2/solid/**', '**/l3/solid/**'] })`
- Shared data layer is identical to L1/L2 — no new data files

## CSS Architecture

| File | Used By |
|------|---------|
| `public/qscrape.css` | Root and level index pages (`/`, `/l1/`, `/l2/`, `/l3/`) |
| `public/global.css` | L1 site pages (taxes, news) — legacy ASP.NET Web Forms style |
| `src/styles/l2/*.css` | L2 components — design tokens per site, shared across all 3 frameworks |
| `src/styles/l3/*.css` | L3 islands — design tokens per site, no global resets |

## Project Structure

```
src/pages/
  index.astro              — Root index (QScrape home)
  l1/
    index.astro            — L1 level index
    news/                  — Mountainhome Herald
    scoretap/              — ScoreTap
    taxes/                 — Eldoria Registry of Deeds
    eshop/                 — VaultMart
  l2/
    index.astro            — L2 level index
    react/news/            — Mountainhome Herald (React) — 6 shells
    react/eshop/           — VaultMart (React) — 5 shells
    react/scoretap/        — ScoreTap (React) — 3 shells
    react/taxes/           — Eldoria Registry (React) — 5 shells
    vue/news/              — Mountainhome Herald (Vue) — 6 shells
    vue/eshop/             — VaultMart (Vue) — 5 shells
    vue/scoretap/          — ScoreTap (Vue) — 3 shells
    vue/taxes/             — Eldoria Registry (Vue) — 5 shells
    svelte/news/           — Mountainhome Herald (Svelte) — 6 shells
    svelte/eshop/          — VaultMart (Svelte) — 5 shells
    svelte/scoretap/       — ScoreTap (Svelte) — 3 shells
    svelte/taxes/          — Eldoria Registry (Svelte) — 5 shells
src/components/l2/
  news/{react,vue,svelte,solid}/       — News islands (all frameworks)
  eshop/{react,vue,svelte,solid}/      — Eshop islands (all frameworks)
  scoretap/{react,vue,svelte,solid}/   — ScoreTap islands (all frameworks)
  taxes/{react,vue,svelte,solid}/      — Taxes islands (all frameworks)
src/data/
  news/articles.ts         — Shared news data (L1 re-exports from here)
  eshop/products.ts        — Shared eshop data (L1 re-exports from here)
  scoretap/data.ts         — Shared scoretap data
  taxes/deeds.ts           — Shared taxes data
src/styles/l2/
  news.css / eshop.css / scoretap.css / taxes.css  — Design tokens
src/styles/l3/
  news.css / eshop.css / scoretap.css / taxes.css  — Design tokens (no global resets)
public/
  qscrape.css              — Shared dark theme for index pages
  global.css               — Legacy ASP.NET style for L1 site pages
  *.pdf                    — Tax record PDFs (15 deed/mortgage/lien files)
  how-to/                  — How-to guide PDFs (8 files, one per index type)
  l3/
    index.astro            — L3 level index
    news/
      index.astro          — News home (4 islands)
      articles/index.astro — Articles listing (4 islands)
      article/[id].astro   — Article detail (4 islands, getStaticPaths)
    eshop/
      index.astro          — Eshop home (4 islands)
      product/[sku].astro  — Product detail (4 islands, getStaticPaths)
    scoretap/
      index.astro          — ScoreTap home (4 islands)
      teams/index.astro    — Teams listing (4 islands)
    taxes/
      index.astro          — Taxes home (4 islands)
      viewer/[id].astro    — Deed viewer (4 islands, getStaticPaths)
src/components/l3/
  react/{news,eshop,scoretap,taxes}/   — React islands (CSS-assembled text)
  vue/{news,eshop,scoretap,taxes}/     — Vue islands (pseudo-element content)
  svelte/{news,eshop,scoretap,taxes}/  — Svelte islands (decoy overlay)
  solid/{news,eshop,scoretap,taxes}/   — Solid islands (canvas-rendered text)
src/
  middleware.ts            — Alias/resolver middleware for URL routes
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CascadingLabs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CascadingLabs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
