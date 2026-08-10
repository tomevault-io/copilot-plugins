## cardd-cc

> SVG badge service for the web, deployed to Cloudflare Workers.

# cardd

SVG badge service for the web, deployed to Cloudflare Workers.

## Stack

- **Runtime:** Cloudflare Workers (via Wrangler)
- **Framework:** Hono (SSR, routing, `hono/css` for scoped CSS-in-JS)
- **Rendering:** satori (JSX → SVG)
- **Caching:** KV — full URL key (prefixed `svg:`), 1hr TTL, check-first
- **Fonts:** Datatype (bundled), Google Fonts (fetched on-demand → KV cache)
- **Rate limiting:** `hono-rate-limiter` using Workers Rate Limiting API
- **Package manager:** pnpm

## Routes

| Route | Example | Source |
|---|---|---|
| `/badge/*` | `/badge/license-MIT?bg=555` | Static label\|value |
| `/npm/v/:pkg` | `/npm/v/express` | registry.npmjs.org |
| `/npm/d/:pkg` | `/npm/d/express` | api.npmjs.org |
| `/gh/stars/:owner/:repo` | `/gh/stars/expressjs/express` | api.github.com |
| `/partial/:badgeId` | `/partial/static` | HTMX fragment for demo panel |
| `/fonts/Datatype-Regular.ttf` | (internal) | Bundled font asset |
| `/` | Landing page | SSR via `hono/jsx-renderer` |

Static badges use the path segment after `/badge/`, splitting on the first `-` to separate label from value.

## Architecture

Badge types are classes extending an abstract `Badge` base class (`src/badges/types.ts`). Each badge defines its `id`, `title`, `path`, `examplePath`, `fetch()` method, `onError()` handler, and optional `pathParams`/`demoPresets`. The app iterates `BADGES` from `src/badges/index.ts` to register routes and partial fragment endpoints.

The landing page is an SSR-rendered Hono JSX page with a sidebar, main content, and demo panel. The demo panel loads badge previews via HTMX fragments served at `/partial/:badgeId`.

## CSS customization (query params)

| Param | Example | Default |
|---|---|---|
| `bg` | `bg=2ea44f` | `#2d2d2e` (dark) / `#f5f5f5` (light) |
| `color` | `color=fff` | `#fff` (dark) / `#333` (light) |
| `radius` | `radius=8` | `4` |
| `border` | `border=2+solid+333` | none |
| `borderColor` | `borderColor=f00` | `#2d2d2e` (dark) / `#d0d0d0` (light) |
| `font` | `font=Inter` | `Datatype` |
| `theme` | `theme=light` | `dark` |

`bg` and `color` accept named CSS colors (e.g. `red`, `dodgerblue`) in addition to hex values. `theme` can be `dark`, `light`, or `auto`. `auto` renders both themes and uses `prefers-color-scheme` CSS inside the SVG to toggle at runtime. When `theme=auto`, explicit `?bg`/`?color` are ignored.

Plus `+` is used as the border shorthand delimiter. Hex values omit `#`.

## Design decisions

- **Single background, pipe separator** — ASCII-aesthetic label|value layout, not shields.io's two-tone split.
- **satori over manual SVG** — Enables JSX-based layouts with flexbox, dynamic text measurement, and arbitrary CSS without manual character-width heuristics.
- **Datatype bundled, Google Fonts fetched** — Datatype is a GitHub-hosted font not served by the Google Fonts CSS API, so it's imported as a binary module (configured via `rules` in `wrangler.json`). Other fonts fetch from Google Fonts on-demand and cache in KV for 30 days.
- **KV caching by full URL** — SVGs are cached including query params. 1hr TTL. Check-first strategy. No stale-while-revalidate yet.
- **Error handling** — Stale KV entries are served if they exist. Otherwise descriptive error badges render (e.g. `pkg | not found`, `npm | timeout`). Render failures produce a generic `error | render failed` badge or fall through to a 500 text response.
- **Rate limiting** — Global rate limiter (100 req/min per IP) using Workers Rate Limiting API, with `/` and `/fonts/` routes skipped.
- **`moduleResolution: "Node16"`** — Required for proper ESM module resolution with `.js` extension imports mapping to `.ts` source files.

## Project structure

```
src/
├── index.tsx              # Hono app, route registration, font route, landing page
├── renderer.tsx           # satori JSX badge component + render functions
├── types.d.ts             # TypeScript declarations for .ttf imports
├── badges/
│   ├── index.ts           # BADGES registry
│   ├── types.ts           # Abstract Badge base class
│   ├── static.ts          # StaticBadge (/badge/*)
│   ├── npm-version.ts     # NpmVersionBadge (/npm/v/:pkg)
│   ├── npm-downloads.ts   # NpmDownloadsBadge (/npm/d/:pkg)
│   └── gh-stars.ts        # GhStarsBadge (/gh/stars/:owner/:repo)
├── pages/
│   └── LandingPage.tsx    # SSR landing page with CSS variables, light/dark themes
├── components/
│   ├── Header.tsx         # Header with nav links
│   ├── Sidebar.tsx        # Left sidebar navigation
│   ├── MainContent.tsx    # Documentation content (overview, badges table, customization, API ref)
│   ├── DemoPanel.tsx      # Right panel with HTMX-powered badge demo forms
│   ├── BadgePreview.tsx   # Simple <img>-based badge preview
│   ├── CodeBlock.tsx      # <pre><code> wrapper
│   └── icons/
│       └── Github.tsx     # GitHub SVG icon
└── utils/
    ├── badge.ts           # handleBadge / handleError — shared render+KV logic
    ├── cache.ts           # KV read/write wrapper, Env type, cacheKey helper
    ├── css.ts             # URL query param → BadgeCSS parsing (DARK/LIGHT presets)
    └── fonts.ts           # Font loading (bundled Datatype + Google Fonts)
fonts/
└── Datatype-Regular.ttf
public/
└── client.js              # Client-side JS
wrangler.json              # Worker config, KV bindings, rate limiter, asset config
tsconfig.json
```

## KV namespaces

| Binding | Production ID | Preview ID | Purpose |
|---|---|---|---|
| `CARDD_KV` | `6cc8e4b4...` | `5bb93eee...` | SVG cache + font cache |

## Environment variables

| Variable | Type | Purpose |
|---|---|---|
| `CARDD_KV` | `KVNamespace` | SVG and font caching |
| `BADGE_RATE_LIMITER` | `RateLimit` | Global rate limiter (100 req/min) |
| `GITHUB_TOKEN` | `string?` | Optional GitHub PAT for higher API rate limits |

## Commands

```
pnpm dev          # wrangler dev
pnpm deploy       # wrangler deploy
pnpm typecheck    # tsc --noEmit
```

---
> Source: [stordahl/cardd.cc](https://github.com/stordahl/cardd.cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
