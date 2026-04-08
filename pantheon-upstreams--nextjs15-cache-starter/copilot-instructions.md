## nextjs15-cache-starter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start development server
npm run build        # Production build
npm run start        # Start production server
npm run lint         # Run ESLint
```

## Architecture

This is a **Next.js 15** app (App Router) that demonstrates custom cache handlers via the `@pantheon-systems/nextjs-cache-handler` package. It uses Tailwind CSS v4 for styling.

### Cache System

The cache handler is configured in `cache-handler.mjs` and wired into Next.js via `next.config.mjs` (`cacheHandler` + `cacheMaxMemorySize: 0` to disable in-memory caching). The handler auto-selects between GCS and file-based storage (GCS is automatically configured when deployed on Pantheon).

Cache stats and clearing are exposed via `@pantheon-systems/nextjs-cache-handler`'s `getSharedCacheStats` and `clearSharedCache` exports, used in `app/api/cache-stats/route.ts`.

### Surrogate Keys

`next.config.mjs` sets `Surrogate-Key` headers on responses for CDN cache invalidation. Routes under `/blogs` get structured keys (e.g., `blog-:slug`, `blog-index-tag-:tag`); other routes get `unknown`.

### Middleware

`middleware.ts` adds `x-middleware-executed` and `x-intercepted-path` debug headers. **Middleware is incompatible with `NEXT_PRIVATE_MINIMAL_MODE=1`** — it causes empty responses.

### Key Data Flow

- `lib/blogService.ts` fetches from WordPress REST API (default: `developer.wordpress.org/news`) with various Next.js cache strategies (`no-store`, `force-cache`, `revalidate`, tagged)
- Blog pages (`app/blogs/`) use ISR: list page revalidates every 300s, detail pages every 600s
- `app/blogs/[slug]/page.tsx` uses `generateStaticParams` for SSG
- `BlogPost` type is defined in `app/blogs/page.tsx` and imported elsewhere

### API Routes

| Endpoint | Purpose |
|---|---|
| `GET /api/cache-info` | Lists available cache strategy endpoints |
| `GET/DELETE /api/cache-stats` | View/clear cache statistics |
| `POST /api/revalidate` | Invalidate a cache tag (`{ "tag": "..." }`) |
| `GET /api/revalidate?tag=...` | Same via query param |
| `POST /api/purge` | Purge surrogate keys (`{ "surrogateKeys": [...] }`) — requires `WEBHOOK_SECRET` |
| `POST /api/nuke-cdn` | Purge CDN cache via outbound proxy |
| `/api/posts/no-cache` | Fetch demo: `cache: 'no-store'` |
| `/api/posts/force-cache` | Fetch demo: `cache: 'force-cache'` |
| `/api/posts/revalidate` | Fetch demo: `next: { revalidate: 60 }` |
| `/api/posts/with-tags` | Fetch demo: tagged cache with 5min TTL |

### Environment Variables

- `WORDPRESS_API_URL` — WordPress REST API base (default: `https://developer.wordpress.org/news/wp-json/wp/v2`)
- `WEBHOOK_SECRET` — Shared secret for authenticating `/api/purge` requests (required)

## Code Style Guidelines

- **Small, focused functions** with single responsibilities and descriptive names
- **Early returns / guard clauses** over nested if statements — keep code flat
- **Targeted try-catch blocks** — catch specific errors close to where they occur, not broad catch-alls around large blocks
- **No unnecessary nesting** — break nested logic into separate functions
- **Document function purpose** with JSDoc comments explaining the objective

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pantheon-upstreams)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pantheon-upstreams)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
