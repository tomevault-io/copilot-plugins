## woodlands-family-practice

> A Next.js site optimised for SEO. Pages are pre-rendered to static HTML at build time.

# Project Overview


A Next.js site optimised for SEO. Pages are pre-rendered to static HTML at build time.


---


# Voice — read before writing any content


When writing **any blog post, service page, or customer-facing copy**, read the files in `./references/`:


| File | What it is |
|------|-----------|
| `references/voice.md` | Writing style, sentence rhythm, vocabulary, formatting, anti-patterns |
| `references/humour.md` | How the brand handles humour |
| `references/stats.md` | Canonical real numbers — pricing, response times, jobs, reviews |
| `references/stories.md` | Recurring anecdotes the brand uses |
| `references/opinions.md` | Hot takes and strong opinions backed by numbers |


**Content rules:**


- Never use AI-tell phrases (e.g. "unlock", "leverage", "seamless", "world-class", "in today's fast-paced world"), exclamation marks, or emojis
- Start with the answer; add context after
- Use real numbers from `stats.md`; round for readability unless precision is essential
- One story per post max (from `stories.md`, don't invent new ones)
- One strong opinion per post max (from `opinions.md`, backed by a number)
- Tell people when NOT to hire you — biggest voice tell
- **Rule 1 — Image limits:** Blog posts must use a maximum of 2 images pulled from Pexels. A third image is only permitted for posts over 1,500 words. Never repeat the same image twice in the same post. Place one image near the top as the hero image and one image roughly halfway through the post above a H2 heading.
- **Rule 2 — Research transparency:** Never explain your research process, methodology, or sources in the body of the blog post. Never reference competitor pages, ranking guides, or SEO research in the content itself. The research informs the writing — it is never mentioned in the writing.


Before shipping any writing, re-read `references/voice.md` sections on structural rules and banned phrases, and remove anything that matches.


---


# On-page SEO


When generating or editing a blog post, read `on-page-seo.md` at the root. Every item applicable to the page type must be satisfied.


Required for every long-form post:
- FAQ section with FAQPage schema (JSON-LD)
- Breadcrumbs + BreadcrumbList schema
- Author byline + Person schema
- Table of contents with anchor links
- 3–5 internal links, 2–3 external links to authoritative sources
- Open Graph + Twitter Card meta
- Length within 20% of SERP top-3 for the target keyword


---


# Technical SEO


Site-wide:
- `public/sitemap.xml` — canonical URL list for crawlers (update `lastmod` when pages materially change)
- `public/robots.txt` — allows all crawlers; references the sitemap URL (must match production hostname)
- Canonical URLs on every page (via `metadata.alternates.canonical`)
- Open Graph images (1200×630) — use `/public/og/*.png` where available, or approved external image URLs
- Image width/height attributes for CLS
- Semantic HTML5 (`<header>`, `<nav>`, `<main>`, `<article>`, `<footer>`)
- Static pre-rendering — `output: 'export'`
- Mobile viewport — set in `app/layout.tsx`


---


# Design


Premium, modern, elegant. Subtle animations, proper spacing, clear visual hierarchy. No emoji icons. No generic gradients. One accent colour (pick one and stick to it).


---


# Development Rules


**Rule 1: Always read first** — before any action, read `CLAUDE.md`.


**Rule 2: Define before you build** — no code before spec approval.


**Rule 3: Look before you create** — check existing files before creating new ones.


**Rule 4: Test before you respond** — run `npm run build` before saying "done".


**Core Rule** — do exactly what is asked. Nothing more, nothing less.


---


# Tech Stack


- **Language:** TypeScript
- **Framework:** Next.js 14 (App Router)
- **Rendering:** Static Site Generation via `output: 'export'`. `out/` is the deployable.
- **Styling:** Tailwind CSS
- **Content:** Route-level content in `app/**` plus shared data in `lib/**`. No database.
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


# Organisation Rules


- One component per file
- Shared components live in `/components/`
- Page-specific content lives with routes in `app/**` and shared structures in `lib/**`
- Don't create new top-level folders without asking


---


# Testing


Before marking any task done:
- `npm run build` completes with no errors
- Every route shows `○ (Static)` in the build log
- **View-source check:** the HTML contains the actual rendered content and any JSON-LD schema
- **Voice check** (for content changes): re-read `references/voice.md` → "Tells that it's AI-written" and delete anything that matches


Never say "done" if the build is failing, there are console errors, the voice reads as AI, or the feature hasn't been tested in the browser.


---


# Scope


Only build what's requested. If anything is unclear, ask before starting.

---
> Source: [jimmymilligan93-maker/woodlands-family-practice](https://github.com/jimmymilligan93-maker/woodlands-family-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
