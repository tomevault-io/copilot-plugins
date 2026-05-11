## seo-brief

> **Single Next.js project. Six versions as pages.** Each version demonstrates one chapter of the SEO workflow.

# Project Overview

**Single Next.js project. Six versions as pages.** Each version demonstrates one chapter of the SEO workflow.

| Route | Chapter | Purpose |
|-------|---------|---------|
| `/` | — | Landing page, links to all 6 versions |
| `/v1` | Ch 2 | Scaffolded site — Claude Code built it from a one-line prompt |
| `/v2` | Ch 3a | AI blog slop — blog post written without voice files |
| `/v3` | Ch 3b | Voice-injected — same topic, written against `references/` |
| `/v4` | Ch 5 | Landing page — toilet repair service page in voice |
| `/v5` | Ch 6a | On-page SEO — v3 post + FAQ/Breadcrumb/Article/Person schema, TOC, author card, internal links |
| `/v6` | Ch 6b | Technical SEO — v5 + sitemap.xml, robots.txt, OG images, Core Web Vitals |

Site is **Static Site Generation** via `output: 'export'`. Every route pre-rendered to HTML at build time.

---

# Voice — read before writing any content

When writing **any blog post, service page, or customer-facing copy**, read the files in `./references/`:

| File | What it is |
|------|-----------|
| `references/voice.md` | Marco's writing style, sentence rhythm, vocabulary, formatting, anti-patterns |
| `references/humour.md` | How Marco handles humour (dry, industry-aimed, never puns) |
| `references/stats.md` | Canonical real numbers — pricing, response times, jobs, reviews |
| `references/stories.md` | Recurring anecdotes (Windsor garage, fat-in-the-candle, tree roots) |
| `references/opinions.md` | Hot takes on the industry, hot water systems, pricing, older homes |

**Content rules derived from these files:**

- Never use: "unlock", "leverage", "seamless", "world-class", "in today's fast-paced world", exclamation marks, emojis
- Start with the answer; add context after
- Use real numbers from `stats.md`, never round
- One story per post max (from `stories.md`, don't invent new ones)
- One strong opinion per post max (from `opinions.md`, backed by a number)
- Tell people when NOT to hire us — biggest voice tell

Before shipping any writing, re-read `references/voice.md` → "Tells that it's AI-written" and delete anything that matches.

---

# On-page SEO — for v5+ content

When generating or editing **blog posts for v5 or v6**, read `on-page-seo.md` at the root. Every item applicable to the page type must be satisfied.

Key v5/v6 additions on top of v3:
- FAQ section with FAQPage schema (JSON-LD)
- Breadcrumbs + BreadcrumbList schema
- Author byline + Person schema
- Table of contents with anchor links
- 3–5 internal links, 2–3 external links to authoritative sources
- Open Graph + Twitter Card meta
- Length within 20% of SERP top-3 for the target keyword

---

# Technical SEO — v6 only

v6 adds site-wide technical SEO:
- `app/sitemap.ts` — auto-generated sitemap covering all routes
- `app/robots.ts` — allows all crawlers, points to sitemap
- Canonical URLs on every page (via `metadata.alternates.canonical`)
- Open Graph images (1200×630) — `/public/og/*.png`
- Image width/height attributes for CLS
- Semantic HTML5 (`<header>`, `<nav>`, `<main>`, `<article>`, `<footer>`)
- Static pre-rendering — `output: 'export'`
- Mobile viewport — set in `app/layout.tsx`

---

# Design

Premium, modern, elegant. Subtle animations, proper spacing, clear visual hierarchy. No emoji icons. No generic gradients. One accent colour (indigo).

---

# Development Rules

**Rule 1: Always read first** — before any action, read `CLAUDE.md` and `project_specs.md`.

**Rule 2: Define before you build** — no code before spec approval.

**Rule 3: Look before you create** — check existing files before creating new ones.

**Rule 4: Test before you respond** — run `npm run build` before saying "done".

**Core Rule** — do exactly what is asked. Nothing more, nothing less.

---

# Tech Stack

- **Language:** TypeScript
- **Framework:** Next.js 15 (App Router)
- **Rendering:** Static Site Generation via `output: 'export'`. `out/` is the deployable.
- **Styling:** Tailwind CSS
- **Content:** Flat TypeScript files in `/content/*.ts`. No database.
- **Deployment:** Vercel

**SSG constraints — do NOT break these:**
- No `cookies()`, `headers()`, or `searchParams` in server components
- No `fetch(..., { cache: 'no-store' })` or `export const dynamic = 'force-dynamic'`
- No runtime API routes
- Dynamic routes (`[slug]`) must implement `generateStaticParams`
- All data fetched at **build time**, not request time

---

# Running the Project

1. `npm install`
2. `npm run dev` — opens on `http://localhost:3000`
3. To ship: `npm run build` → the `out/` directory is the deployable site

---

# File Structure

```
app/
  layout.tsx             shared — fonts, header, footer, sitewide schema
  page.tsx               landing page — links to all 6 versions
  globals.css
  sitemap.ts             (v6 technical SEO artifact)
  robots.ts              (v6 technical SEO artifact)
  v1/page.tsx            scaffolded homepage
  v2/page.tsx            AI-slop blog post
  v3/page.tsx            voiced blog post (same URL pattern as v2)
  v4/page.tsx            toilet repair landing page
  v5/page.tsx            voiced post + on-page SEO
  v6/page.tsx            voiced post + technical SEO scorecard
components/
  site/                  Header, Footer, Logo, VersionSwitcher
  home/                  Hero, FeatureGrid, HowItWorks, Testimonials, ServiceAreas, FinalCta
  blog/                  BlogBody, VersionBanner, Breadcrumbs, TableOfContents, AuthorCard
  seo/                   LocalBusinessJsonLd, BlogJsonLd, TechSeoScorecard
content/
  blog-ai-slop.ts        v2 post content
  blog-voiced.ts         v3/v5/v6 post content (with faqs[] + author object)
  toilet-repair.ts       v4 service content
lib/
  business.ts            single source of truth for the business
  markdown.ts            minimal md renderer
  seo.ts
references/              voice, humour, stats, stories, opinions (read before writing)
on-page-seo.md           checklist v5/v6 must satisfy
```

**Organisation rules:**
- One component per file
- Don't create new top-level folders without asking
- Shared components live in `/components/`; per-version pages import what they need

---

# Testing

Before marking any task done:
- `npm run build` completes with no errors
- Every route shows `○ (Static)` in the build log
- **View-source check:** open `/v2/`, `/v3/`, `/v6/` — the HTML contains the actual rendered content and, for v5/v6, the JSON-LD schema
- **Voice check** (for content changes): re-read `references/voice.md` → "Tells that it's AI-written" and delete anything that matches

Never say "done" if the build is failing, there are console errors, the voice reads as AI, or the feature hasn't been tested in the browser.

---

# Scope

Only build what's requested. If anything is unclear, ask before starting.

---
> Source: [jonocatliff/SEO_brief](https://github.com/jonocatliff/SEO_brief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
