## seanogrady-me

> > Personal website and blog built with Astro, Vue 3 islands, and Astro Content Collections

# seanogrady.me

> Personal website and blog built with Astro, Vue 3 islands, and Astro Content Collections

## Stack

- **Astro 5** — static site with Vue islands for interactivity
- **Vue 3** — interactive components (DarkModeToggle, HomeAvatar, AppNowPlaying)
- **Astro Content Collections** — Markdown blog posts in `src/content/posts/`
- **Tailwind CSS 4** via Vite plugin (not PostCSS) + Radix Colors (`sage`, `jade`)
- **@astrojs/cloudflare** — adapter for server-rendered API routes
- **astro-icon** — Iconify-powered, build-time SVG inlining

## Commands

```bash
pnpm dev        # localhost:4321 (assume already running)
pnpm build      # production build (static + server routes)
pnpm preview    # preview production build
```

## Key Conventions

- **Astro components** (`.astro`) for static content, **Vue components** (`.vue`) for interactivity
- **Vue islands** use `client:load` (above fold) or `client:visible` (below fold)
- **Dates**: store as ISO strings in frontmatter; format with Luxon `DateTime`
- **Blog frontmatter**: use `date` (not `publishedAt`), optional `tags: []`
- **Styling**: Tailwind classes only — no inline styles or custom CSS
- **Radix Colors harmonies**: https://www.radix-ui.com/colors/docs/palette-composition/understanding-the-scale
- **View transitions** via `<ClientRouter />` in BaseLayout

## Gotchas

- Last.fm "Now Playing" requires env vars: `LASTFM_API_KEY`, `LASTFM_USERNAME`
- The `/api/lastfm` endpoint uses `export const prerender = false` for server-side execution
- Vue islands need inline SVGs for icons (astro-icon only works in .astro files)

## Framework Docs

- **Astro**: https://docs.astro.build/llms-full.txt
- **Vue 3**: https://vuejs.org/llms-full.txt

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seanogdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
