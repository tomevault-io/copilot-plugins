## douro-digital

> **Phase:** Implementation — Card Polish + Cursor Fix

# Douro Digital

**Phase:** Implementation — Card Polish + Cursor Fix

## Stack
- Next.js 16 App Router, React 19, TypeScript
- GSAP + ScrollTrigger (scroll-pin animations, carousels)
- CSS Modules (no Tailwind)
- MDX blog via @next/mdx + next-mdx-remote
- Cal.com embed for booking
- Drizzle ORM + node-postgres (PostgreSQL)

## Structure
- `app/(site)/` — public-facing routes (home, about, work, resources, case-studies, book-a-call)
- `app/(admin)/admin/` — auth-gated admin panel (dashboard, analytics, content)
- `app/api/track/` — page view tracking endpoint
- `components/` — shared UI components (including Analytics tracker)
- `data/` — static data files
- `lib/` — utilities, blog helpers, contexts, db client, schema, auth
- `content/` — MDX blog posts
- `public/` — static assets (images, fonts, videos)
- `drizzle/` — migration output directory

## Database
- `lib/db.ts` — Drizzle connection client
- `lib/schema.ts` — pageViews + contentBlocks tables
- `lib/auth.ts` — cookie-based admin auth (env var password)
- `lib/content.ts` — CMS content helpers (getContent, getContentWithFallback, setContent, listContent)
- `drizzle.config.ts` — migration config
- Admin auth: password from ADMIN_PASSWORD env var, httpOnly cookie session
- Analytics: client-side beacon on route change → POST /api/track → pageViews insert

## CMS Admin Panel (Chunk 3)
- `app/api/content/route.ts` — GET (list all blocks), PUT (upsert, auth-protected)
- `app/(admin)/admin/content/page.tsx` — content listing grouped by section
- `app/(admin)/admin/content/[key]/page.tsx` — content editor (loads block by key)
- `app/(admin)/admin/content/[key]/ContentEditor.tsx` — client-side JSON editor form
- `scripts/seed-content.ts` — seeds all data/*.ts exports into contentBlocks (run: `npx tsx scripts/seed-content.ts`)
- Content keys use dot notation: `home.hero`, `about.team.members`, `services`, etc.
- `getContentWithFallback()` gracefully falls back to static data if DB is unavailable (build-safe)
- Pattern proven: `MeetTheTeam` component fetches from DB with static fallback

## Branch
- `copywriting-updates` — restructure + Raw & Real copy integration

## Recent Changes (uncommitted)
- **WorkShowcase** (`components/work/WorkShowcase.tsx` + `.module.css`): Full-bleed stacked pin cards with MagneticCard, hover effects (image zoom, overlay gradient shift, content lift, hover pill, service tags, arrow fill, image filter, focus-visible)
- **Z-index fix**: Changed `totalCards - i` → `i + 1` so later cards stack ON TOP of earlier ones
- **ScrollPrompt** (`components/effects/ScrollPrompt.tsx`): "Scroll to explore" / "Keep scrolling" bouncing chevron — added to WorkShowcase (first card) and WorkGrid (home page)
- **ProjectCard** (`components/home/ProjectCard.tsx` + `.module.css`): Converted from inline styles to CSS Modules. Added matching hover effects (overlay shift, image filter, pill, content lift, arrow icon)
- **CustomCursor** (`components/cursor/CustomCursor.tsx` + `.module.css`): Fixed `revealed` class being wiped by React re-renders (now uses React state). Added SVG trailing red line (lerp 0.12 factor) + center dot anchor on expanded modes
- **Deleted dead code**: WorkBentoGrid.tsx/css, WorkProjectCard.tsx/css

---
> Source: [DSTARX1/Douro-Digital](https://github.com/DSTARX1/Douro-Digital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
