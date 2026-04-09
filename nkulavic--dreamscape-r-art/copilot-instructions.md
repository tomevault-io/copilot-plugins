## dreamscape-r-art

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Portfolio website for muralist Rachel Dinda (DREAMSCAPER) — a Next.js 15 App Router site deployed on Vercel at dreamscaper.art. Showcases large-scale mural artwork with parallax effects and storytelling.

## Commands

- `npm run dev` — Start dev server (localhost:3000)
- `npm run build` — Production build
- `npm start` — Run production server
- `npm run lint` — ESLint (no --fix script; run `npx eslint --fix` manually)
- No test framework is configured

## Tech Stack

- **Next.js 15** with App Router, React 19, TypeScript 5
- **Tailwind CSS v4** via `@tailwindcss/postcss` (uses `@theme inline` directives, not tailwind.config)
- **Framer Motion 12** for parallax and scroll animations
- **Vercel Postgres** (Neon) via **Drizzle ORM** for structured data
- **Vercel Blob** for media storage (mural images, client logos, videos)
- **Better Auth** for admin authentication (email/password)
- **Groq SDK** (`groq-sdk`) for AI-assisted content generation (openai/gpt-oss-120b model)
- **Sonner** for toast notifications in admin panel
- **Leaflet / React-Leaflet** for interactive mural map
- **React Icons** for iconography
- **UUIDv7** for database primary keys (`uuidv7` package)
- Path alias: `@/*` maps to `./src/*`

## Architecture

### Routing & Pages

All routes live under `src/app/` using file-based App Router conventions. Dynamic route: `/portfolio/[slug]` for individual mural detail pages. Pages are server components by default; interactive components use `"use client"`.

### Data Layer

Content is stored in **Vercel Postgres** (Neon) via **Drizzle ORM**. Media files are served from **Vercel Blob**.

**Database schema** (`src/db/schema.ts`):
- `murals` — Mural entries with UUIDv7 PKs, storytelling fields, blob image URLs, client FK
- `clients` — Client list with slug, logo blob URL, category enum
- `exhibitions`, `festivals`, `publications` — CV data
- `videos` — Behind-the-scenes video collection
- Better Auth tables (user, session, account, verification) — managed by the library

**Data Access Layer** (`src/db/dal.ts`):
- Async query functions: `getAllMurals()`, `getFeaturedMurals()`, `getMuralBySlug()`, `getMuralsBySlugs()`, etc.
- Transform functions convert DB rows to typed interfaces
- All pages fetch via DAL in Server Components, pass data to Client Components as props

**Static TypeScript** (`src/app/data/`):
- `siteConfig.ts` — Global config (artist info, social links, service definitions, credentials)
- `marketing.ts` — Editorial content (audience configs, process steps, investment tiers, FAQs)
- `experience.ts` — Credentials only (exhibitions/festivals/publications moved to DB)

**Admin API routes** (`src/app/api/admin/`):
- CRUD endpoints for murals, clients, experience, videos
- Upload endpoint for Vercel Blob
- AI generation endpoint (`/api/admin/ai/generate`) for content assistance
- All routes require authenticated admin session

**AI Content Generation** (`src/lib/groq.ts`, `src/app/api/admin/ai/generate/route.ts`):
- **Groq SDK** (`groq-sdk`) with `openai/gpt-oss-120b` model for creative content
- Admin panel features AI-assisted generation for mural storytelling fields
- Individual generation: Icon-only sparkle buttons for title, description, keywords, artistNote, inspiration, process, impact
- Bulk generation: "Generate All Content with AI" button for all storytelling fields at once
- Uses Sonner toast notifications for error handling
- Temperature: 0.6, max_tokens: 500, top_p: 0.9

### Component Organization

- `components/layout/` — Header (fixed, scroll-aware with mobile drawer) and Footer
- `components/ui/` — ParallaxHero, ParallaxSection, VideoPlayer, ThemeSelector, MuralMap, InstagramFeed
- `components/seo/` — JsonLd structured data (Person, LocalBusiness, WebSite, CollectionPage schemas)

### Design System

Color tokens and animations are defined as CSS custom properties in `src/app/globals.css` (40+ variables). Key palette: ocean blues (primary), coral/orange (accent), teal (secondary). Three font families: Bebas Neue (display), Montserrat (headings), Inter (body). Component classes (`.card`, `.btn-primary`, `.btn-secondary`, `.btn-outline`) are defined there too.

### Parallax Pattern

ParallaxHero and ParallaxSection use Framer Motion's `useScroll` + `useTransform` hooks for scroll-based parallax. ParallaxHero supports configurable height and overlay opacity. This pattern was ported from a sibling project (destinyblazek).

### Image Handling

All media is served from **Vercel Blob** (`*.public.blob.vercel-storage.com`). Images are uploaded via the admin panel and stored with deterministic paths. Next.js `<Image>` with `remotePatterns` configured for blob storage, squarespace-cdn.com, and myportfolio.com.

### Authentication & Admin

- **Better Auth** with email/password provider and admin role
- Auth config: `src/lib/auth.ts` (server), `src/lib/auth-client.ts` (client)
- API route: `src/app/api/auth/[...all]/route.ts`
- Middleware: `src/middleware.ts` — protects `/admin/*` routes
- Admin panel: `src/app/admin/` — full CRUD for all content types
- Admin credentials: seeded via `src/db/seed.ts`

## Deployment

Vercel with auto-deploy from GitHub master branch. Security headers (X-Frame-Options, X-Content-Type-Options, Referrer-Policy) configured in `vercel.json`. Redirect: `/home` → `/`.

### Manual Deploy Trigger

To trigger a production deployment manually (bypassing git push):

```bash
curl -X POST https://api.vercel.com/v1/integrations/deploy/prj_OfYPlw7jJasdxfm26PbiPXMRFZIz/Iavom6qSdp
```

Or use the Vercel CLI:

```bash
vercel --prod --yes
```

## Pending Work

### Website Cleanup & Optimization
- Remove old static images/videos from repo (public/images/, public/videos/)
- Lighthouse performance audit & fixes
- WCAG AA accessibility audit & fixes
- High-resolution image swap

### Website Features
- Lightbox gallery for mural detail pages
- HubSpot CRM scheduling integration (contact form is placeholder)
- Instagram feed integration (Behold.so)
- Contact form — connect to HubSpot or email

### Business Listings (Rachel)
- Google Business Profile
- Apple Maps (Apple Business Connect)
- Yelp listing
- Social media profile audit

## Teamwork Integration

Project is managed in Teamwork (project ID: 753246, company: Dreamscape-R).

### Plan File Sync
When entering plan mode, always sync the plan file to Teamwork as a notebook:
- **Notebook ID**: 417837 ("Development Plan — CLAUDE.md")
- **On plan creation/update**: Use `teamwork_notebooks` action `update` (notebookId: 417837) to sync the plan file content
- **On session start**: If the plan file is empty or missing, check the Teamwork notebook for the latest version
- This ensures development context is never lost between Claude Code sessions

### Task Tracking
- All development tasks are tracked in Teamwork project 753246
- When starting work on a task, move it to "In Progress" via workflow
- When completing work, mark the task complete and move to "Done"
- Create new tasks in Teamwork for any new work items discovered during development

### Key IDs
- **Project**: 753246
- **Plan Notebook**: 417837
- **Nick Kulavic**: 152544
- **Rachel Dinda**: 505347
- **Claude Code**: 498872

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nkulavic)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nkulavic)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
