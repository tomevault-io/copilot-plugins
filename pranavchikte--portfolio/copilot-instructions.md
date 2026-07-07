## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server with Turbopack (http://localhost:3000)
npm run build    # Production build with Turbopack
npm run start    # Start production server
npm run lint     # Run ESLint
```

No test suite is configured.

## Architecture

**Next.js 16 App Router** portfolio site using React 19, TypeScript, and Tailwind CSS v4.

### Pages

| Route | File | Notes |
|-------|------|-------|
| `/` | `app/page.tsx` | Single-page layout with 6 scroll sections |
| `/projects/[slug]` | `app/projects/[slug]/page.tsx` | Dynamic project case study |
| `/resume` | `app/resume/page.tsx` | PDF viewer page |
| `/opengraph-image` | `app/opengraph-image.tsx` | OG image route |

### Data layer

All content is static — no CMS or API calls at runtime.

- **`lib/projects.ts`** — single source of truth for all project data. The `Project` type defines the full shape. `getFeaturedProjects()` and `getProjectBySlug()` are the only access functions.
- **`lib/site.ts`** — global constants: name, email, URLs, social links, resume PDF path. `formspreeEndpoint` reads from `NEXT_PUBLIC_FORMSPREE_ENDPOINT`.
- **`lib/utils.ts`** — `cn()` helper (clsx + tailwind-merge).

### Design system

The homepage (`app/page.tsx`) uses **inline style tokens** directly rather than CSS variables — colors like `#F8F7F3`, `#1A1918`, `#BF5C1A` are hardcoded constants at the top of the file (`BORDER`, `SERIF`). This is intentional for the editorial design.

`app/globals.css` defines CSS custom properties (`--primary`, `--background`, etc.) mapped into Tailwind v4 via `@theme inline`. These are used by secondary pages (`/resume`, `/projects/[slug]`) and shadcn/ui components.

**Fonts:**
- `--font-geist-sans` / `--font-geist-mono` — body text
- `--font-playfair` (Playfair Display) — headings, serif accents (`SERIF` constant in page.tsx)

**Accent color:** `#BF5C1A` (warm terracotta/orange)

### Components

`components/ui/` — shadcn/ui primitives (button, card, input, label, textarea, avatar). Used primarily on secondary pages. The homepage avoids these in favor of raw HTML with inline styles.

`hooks/` — small utility hooks: `use-animation-settings`, `use-in-view`, `use-outside-click`.

### Navigation

The homepage uses `IntersectionObserver` to track the active section and update the left rail label. Sections are identified by `id` attributes matching the `sections` array in `page.tsx`.

### Adding a project

Edit the `projects` array in `lib/projects.ts`. The `order` field controls display order. The homepage shows only the top 2 (`getFeaturedProjects().slice(0, 2)`); the slug page renders all details.

### Environment variables

| Variable | Purpose |
|----------|---------|
| `NEXT_PUBLIC_FORMSPREE_ENDPOINT` | Contact form submission endpoint |

---
> Source: [Pranavchikte/Portfolio](https://github.com/Pranavchikte/Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
