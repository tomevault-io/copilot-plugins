## solopass

> Auto-generated from all feature plans. Last updated: 2025-11-21

# StackPass Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-11-21

## Active Technologies
- TypeScript 5.8 / Next.js 16.0.1 (App Router with Turbopack) + React, NextAuth v5, Drizzle ORM, PostgreSQL, shadcn/ui (assumed) (004-member-directory)
- PostgreSQL (existing database with users, devcards, achievements tables) (004-member-directory)
- TypeScript 5.8.3, Next.js 16.0.1 (App Router with Turbopack), React 19.2.0, Node ES2017 + Stripe SDK v17.7.0, Drizzle ORM v0.38.4, NextAuth v5.0.0-beta.25, Resend v6.4.2, React Email v4.0.16, Inngest v3.34.5 (005-premium-stripe)
- PostgreSQL via Neon serverless (@neondatabase/serverless v0.10.4), Drizzle ORM for schema managemen (005-premium-stripe)

- TypeScript 5.8 / Next.js 16.0.1 (App Router with Turbopack) (001-devcard-platform)
- SEO: Next.js Metadata API, @vercel/og, Schema.org JSON-LD (003-seo-optimization)

## Project Structure

```text
src/
├── app/                    # Next.js App Router pages
│   ├── (website-layout)/   # Public pages (with SEO metadata)
│   ├── (public)/           # Profile pages (with SEO metadata)
│   ├── (in-app)/           # Authenticated app pages
│   ├── sitemap.ts          # Dynamic sitemap generation
│   ├── robots.ts           # Robots.txt configuration
│   └── layout.tsx          # Root layout with base metadata
├── components/
│   └── seo/                # SEO components (StructuredData, Breadcrumbs)
├── lib/
│   ├── seo/                # SEO utilities (metadata, structured-data)
│   └── config.ts
├── db/
│   ├── schema/
│   └── queries/
│       └── seo.ts          # Sitemap queries
tests/
```

## Commands

npm test && npm run lint

## Code Style

TypeScript 5.8 / Next.js 16.0.1 (App Router with Turbopack): Follow standard conventions

## SEO Guidelines (003-seo-optimization)

- **Metadata**: Use Next.js Metadata API with `generateMetadata()` for dynamic pages
- **OG Images**: Generate via `/api/og/*` endpoints using @vercel/og (1200x630px)
- **Structured Data**: JSON-LD via `<StructuredData>` component (Organization, Event, Person, BreadcrumbList)
- **Sitemap**: Auto-generated at `/sitemap.xml`, includes hackathons, profiles, blog, docs (ISR: 1 hour)
- **Robots.txt**: Disallow `/app/`, `/api/`, `/admin/`, `/super-admin/`
- **Performance**: Maintain existing ISR (hackathons: 600s browse, 60s detail), use priority for above-fold images
- **Error Pages**: Custom 404 (`not-found.tsx`) and 500 (`error.tsx`) with StackPass branding

## Recent Changes
- 005-premium-stripe: Added TypeScript 5.8.3, Next.js 16.0.1 (App Router with Turbopack), React 19.2.0, Node ES2017 + Stripe SDK v17.7.0, Drizzle ORM v0.38.4, NextAuth v5.0.0-beta.25, Resend v6.4.2, React Email v4.0.16, Inngest v3.34.5
- 004-member-directory: Added TypeScript 5.8 / Next.js 16.0.1 (App Router with Turbopack) + React, NextAuth v5, Drizzle ORM, PostgreSQL, shadcn/ui (assumed)

- 001-devcard-platform: Added TypeScript 5.8 / Next.js 16.0.1 (App Router with Turbopack)

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Source: [Danproc/SoloPass](https://github.com/Danproc/SoloPass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
