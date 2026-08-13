## landing-site

> TanStack Start landing site - routing, marketing, scroll video, download flow


# Landing Site Rules

## Stack

- **TanStack Start** + **React 19** + **Tailwind 4**
- **Vite 7** + Nitro for SSR
- **shadcn/ui** primitives in `src/components/ui/`

## Routing

- File-based routes in `src/routes/`
- `routeTree.gen.ts` is **generated** - never hand-edit
- See `src/routes/README.md` for TanStack conventions (not Next.js)

## Constants

**Canonical source:** `shared/constants.ts` (see `.cursor/rules/constants-policy.mdc`).

`src/lib/constants.ts` re-exports `@shared/constants`. Marketing copy and tunables live in `shared/constants.ts`:

| Category | Examples |
|----------|----------|
| Hero copy | `LANDING_HERO_*` |
| Features | `LANDING_FEATURES_*` |
| Scroll video | `LANDING_VIDEO_*`, `LANDING_MAIN_OVERLAP_VH` |
| Paths | `DEMO_PATH`, `EXTENSION_ZIP_FILENAME` |
| Favicon | `SITE_FAVICON_PATH`, `SITE_FAVICON_TYPE` |

Do not hardcode strings in components - import via `@/lib/constants`. FAQ/llms body → `ai-content.json`; SEO builders → `seo.ts`. Per-feature bento clips are off when `LANDING_FEATURE_CLIPS_ENABLED` is `false`.

## Key components

| Component | Role |
|-----------|------|
| `LandingHero.tsx` | Headline, CTAs |
| `ScrollVideoShowcase.tsx` | Sticky scroll launch video |
| `FeatureBentoGrid.tsx` | Feature cards |
| `LandingGetStarted.tsx` | Install steps |
| `LandingNav.tsx` | Nav + GitHub link |

## Scroll video architecture

- Runway height: `LANDING_VIDEO_SCROLL_RUNWAY_VH`
- Phases: expand → hold → shrink (see `getVideoFrameStyle`)
- Features overlap: `LANDING_MAIN_OVERLAP_VH` negative margin on `.landing-main`
- Tunables are sensitive - small constant changes have large UX impact

## Download flow

```typescript
// Fetches EXTENSION_ZIP_FILENAME from public root
fetch(`/${EXTENSION_ZIP_FILENAME}`)
```

ZIP must exist - generate with `npm run package:extension`.
Show inline error if 404 (do not `alert()`).

## Demo embed

- Dev: `webAppDevPlugin.ts` proxies `/demo/` to extension Vite
- Prod: static files in `public/demo/` from `npm run build:web`
- Do not edit `public/demo/` directly

## SSR errors

- `src/server.ts` - custom handler
- `errorCapture.ts` - swallowed error recovery
- `__root.tsx` - `ErrorComponent`, favicon links, global CSS

## SEO & AI discoverability

| Artifact | How it is served | Source |
|----------|------------------|--------|
| Meta + JSON-LD | SSR `<head>` on `/` | `src/lib/seo.ts`, `src/routes/index.tsx` |
| `/robots.txt`, `/sitemap.xml` | Static files in `public/` (gitignored) | `scripts/generate-sitemap.mjs` |
| `/llms.txt` | TanStack Start server route + static copy | `src/routes/llms[.]txt.ts`, generate script |
| FAQ UI + schema | Landing page + FAQPage JSON-LD | `src/lib/ai-content.json`, `LandingFaq.tsx` |

**Environment:** `VITE_SITE_URL` (no trailing slash, prefer `https://www…`) - canonical origin for SEO. Set as an environment variable on your hosting provider for production. Apex/HTTP must permanently redirect to that origin (see root `vercel.json` or equivalent host config).

**Content split:**

- Marketing copy → `shared/constants.ts`
- FAQ / llms summary → `src/lib/ai-content.json`
- FAQ link paths → `"path": "/demo/"` in JSON; resolved in `landingFaqContent.ts`

**Generate & test:**

```bash
npm run generate:seo                    # writes public/robots.txt, sitemap.xml, llms.txt
npm run test -- tests/landing/lib/seo.test.ts
npm run dev:web
curl http://localhost:8080/llms.txt
```

Also runs automatically via `predev:web` and `prebuild:web`. Do **not** commit generated `public/*.txt` / `public/*.xml`.

## Styling

- Landing-specific CSS: `src/landing.css`
- Design tokens: `src/lib/designTokens.ts`
- Extension uses separate `ko-` theme - landing does not share CSS

## Verify

```bash
npm run dev:web
# /, /demo/, download, mobile breakpoint
# SEO: curl /robots.txt /sitemap.xml /llms.txt
```

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
