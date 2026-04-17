## aryankeluskar-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `bun run dev` — Start dev server with Turbopack
- `bun run build` — Production build
- `bun run lint` — ESLint

No test suite is configured.

## Architecture

Next.js 15 personal portfolio site (App Router), deployed on Vercel. Uses Tailwind CSS, Radix UI primitives, and framer-motion for animations.

### Key files

- **`src/data/resume.tsx`** — Central data file. All portfolio content (work experience, education, projects, hackathons, publications, social links) is defined here in a single `DATA` export. This is the primary file to edit when updating site content.
- **`src/app/page.tsx`** — Home page. Client component that renders sections by consuming `DATA`.
- **`src/data/blog.ts`** — Blog post processing. Reads MDX files from `content/` directory, parses frontmatter with gray-matter, and renders markdown via unified/remark/rehype pipeline with syntax highlighting (shiki) and KaTeX math support.
- **`src/app/blog/[slug]/page.tsx`** — Dynamic blog post route.

### Blog posts

Add `.mdx` files to the top-level `content/` directory with frontmatter: `title`, `publishedAt`, `summary`, optional `image`.

### Redirects

Short URL redirects are configured in `next.config.mjs` from two sources:
- `src/data/dub.csv` — dub.sh link slugs parsed from CSV
- `src/data/custom.json` — custom slug-to-dub.sh mappings

### UI components

- `src/components/ui/` — shadcn/ui primitives (button, badge, card, avatar, tooltip, separator)
- `src/components/magicui/` — Animation components (blur-fade, dock)
- `src/components/sections/` — Page sections (publications, writing)
- `src/components/` — Domain components (hero-section, project-card, resume-card, hackathon-card, hometown-section, navbar, blueprint-grid)

### Styling

Tailwind CSS with `@tailwindcss/typography`. Theme toggling via `next-themes`. The `cn()` utility in `src/lib/utils.ts` merges Tailwind classes (clsx + tailwind-merge).

### Route handlers

- `/resume` — Redirects to resume PDF
- `/gcsp/*` — Redirect routes

### Images

Remote images served from `alice.aryankeluskar.com`. Uses `next/image` with that hostname configured in `next.config.mjs`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aryankeluskar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
