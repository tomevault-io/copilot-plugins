## learn-programming-easily

> A content management system for writing and publishing programming articles. The owner writes and posts articles; readers browse the public site.

# Project: Programming Articles CMS

A content management system for writing and publishing programming articles. The owner writes and posts articles; readers browse the public site.

## Tech Stack

- **Framework:** Next.js (App Router, TypeScript)
- **Styling:** Tailwind CSS
- **Database:** PostgreSQL + Prisma ORM
- **Content:** Article body (raw MDX) and metadata both stored in PostgreSQL; rendered from the DB
- **Auth:** Auth.js (credentials provider — single admin user)
- **Syntax Highlighting:** Shiki
- **Search:** Fuse.js (client-side)
- **RSS:** `feed` npm package
- **Deployment:** Vercel

## Architecture

- Public pages are statically generated (SSG/ISR) for SEO and performance, and regenerated on-demand via `revalidatePath` when articles change — publishing goes live with no redeploy
- Admin panel is dynamic, protected behind auth
- Article body (raw MDX) lives in PostgreSQL (`Article.body`); there is no `/content` directory
- Article metadata (title, slug, tags, categories, publish status, dates) also lives in PostgreSQL
- Admin panel creates/edits articles directly in the database (body + metadata) — the Vercel filesystem is read-only, so nothing is written to disk at runtime

## Project Structure

- `prisma/` — database schema and migrations
- `src/app/(public)/` — public-facing routes (homepage, articles, tags)
- `src/app/admin/` — protected admin routes (dashboard, CRUD)
- `src/app/api/` — API routes (auth, CRUD, RSS)
- `src/components/` — React components (site/, admin/, article/, mdx/, search/)
- `src/lib/` — utilities (prisma client, auth config, DB queries, MDX rendering, search, RSS)
- `src/types/` — shared TypeScript types

## Conventions

- Use TypeScript for all files
- Use Tailwind CSS for styling — no CSS modules or styled-components
- Use Server Components by default; add `"use client"` only when needed
- Use Server Actions for form mutations instead of API routes where possible
- Article metadata (title, description, tags, categories, published, featured, publishedAt, coverImage) lives as DB columns/relations, set via the admin form — not file frontmatter
- Render MDX bodies with `next-mdx-remote/rsc` + Shiki; `gray-matter` is used only for the admin live preview, not for storage
- Article mutations must call `revalidatePublicSurfaces()` (in `src/lib/actions/article.ts`) so public pages, RSS, and sitemap update without a redeploy
- Prisma client should be a singleton (avoid hot-reload connection exhaustion)
- All public pages should have proper meta tags for SEO
- Dark/light mode support via Tailwind's dark variant

---
> Source: [HtunAungKyaw73/Learn-Programming-Easily](https://github.com/HtunAungKyaw73/Learn-Programming-Easily) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
