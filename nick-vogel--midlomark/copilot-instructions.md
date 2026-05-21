## midlomark

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev              # Start dev server (NODE_OPTIONS=--no-deprecation)
pnpm devsafe          # rm -rf .next && start dev
pnpm build            # Production build
pnpm start            # Start production server
pnpm lint             # ESLint
pnpm seed             # Seed demo content (user, address, pages, posts, categories, nav)
pnpm generate:types   # Regenerate src/payload-types.ts from payload.config.ts
pnpm payload migrate  # Run database migrations
pnpm ci               # payload migrate && build (CI pipeline)
```

No test suite is configured.

## Tech Stack

- **Next.js 15** (App Router) + **React 19** + **TypeScript 5.7**
- **Payload CMS 3.24** with MongoDB (mongoose adapter)
- **Lexical** rich text editor
- **Tailwind CSS 3.4** + SCSS modules for component styles
- **Cloudflare R2** (S3-compatible) for media storage, served via `MEDIA_URL` env var
- **Resend** for email (nodemailer fallback)
- **pnpm** package manager, **Node ^18.20.2 || >=20.9.0**

## Architecture

### App Router Layout

```
src/app/
├── (app)/                        # Public site
│   ├── layout.tsx                # Root layout (Nav, Footer, global settings)
│   ├── (pages)/                  # Page routes
│   │   ├── page.tsx              # Home (fetches slug: 'home')
│   │   ├── [slug]/page.tsx       # Dynamic pages
│   │   ├── blog/                 # Blog listing + [slug] detail + RSS feed
│   │   ├── industries/[slug]/    # Industry pages
│   │   └── search/               # Search page
│   └── components/               # App-level components (RenderBlocks, RichText, Hero, Nav, Footer, etc.)
├── (payload)/                    # CMS admin panel + REST API
│   ├── admin/[[...segments]]/    # Payload admin UI
│   └── api/[...slug]/            # Auto-generated REST API
```

### Payload CMS Collections & Globals

**Collections:** Pages, Posts, Industries, Media, Users, Categories, Addresses, Redirects, EntitySeo, StickyBanners

**Globals:** GlobalSettings (business info, GTM, logos), Nav, Footer

**Plugins:** Form Builder, SEO, Search, S3 Storage, Blur Data URLs

### Block-Based Page Building

Pages/Industries use an array `content` field of blocks. Each block lives in `src/blocks/<BlockName>/`:
- `config.ts` — Payload field schema (defines CMS editor UI)
- `Component/index.tsx` — React rendering component

Available blocks: ContentWithMedia, ContentNoMedia, Cards, CTA, ContentWithMap, ContentWithVideo, ClientLogos, FAQBlock, PainPoints, SponsoredBlock, Steps, StepsWithIcons, Summary, TableOfContents, Form, Feed, Section.

`RenderBlocks` (`src/app/(app)/components/RenderBlocks/index.tsx`) maps `blockType` → component.

### Shared Field Configs

`src/components/` contains reusable Payload field configuration factories (not React components):
- `buttons.ts`, `image.ts`, `slug.ts`, `headerSection.ts`, `positionSelector.ts`, `socialLinks.ts`, `video.ts`

### Access Control

Defined in `src/utilities/permissions.ts`. User roles: `admin`, `user`, `viewer`, `author`.
- `adminPerms` — admin or own user
- `userPerms` — admin/user or own user
- `viewerPerms` — admin/user/viewer or own user
- `fieldAdmin` — admin or own user (field-level)

### Caching & Revalidation

- Global settings fetched via `getCachedGlobal()` using Next.js `unstable_cache`
- Collection `afterChange` hooks call `revalidatePath()` for published documents
- Static generation with `generateStaticParams()` for pages, posts, industries

### Rich Text Rendering

Lexical serialized state is rendered via custom JSX converters in `src/app/(app)/components/RichText/converters/`.

### Forms

Form Builder plugin creates `forms` + `form-submissions` collections. `FormBlock` renders forms using `react-hook-form` and POSTs to `/api/form-submissions`.

### SEO & Structured Data

- SEO plugin generates meta title/description/image per collection doc
- JSON-LD schema markup via `src/app/(app)/components/Schema/`
- Canonical URL support

### Image URLs

Use `getImageUrl()` from `src/utilities/getImageUrl.ts` to resolve media URLs. It accepts a `Media` object, filename string, or null. It prefers `media.url` (populated by the S3 plugin's `generateFileURL`) and falls back to constructing from `MEDIA_URL` + filename. This requires a CDN domain pointing to the S3 bucket — see the **Media Storage (CDN)** section in `README.md` for details and removal steps.

## Key Conventions

- **Path alias:** `@/*` maps to `./src/*`
- **Payload custom components** must be referenced as string paths, not direct imports
- **Auto-generated types:** Always use types from `@/payload-types` for Payload data. Run `pnpm generate:types` after schema changes.
- **CSS variables** defined in `src/app/(app)/global.css` — dark primary (#06410F), light primary (#DDFCE2), accent (#57D9EF)
- **Media images** served via CDN URL generated by the S3 plugin (`generateFileURL`). `MEDIA_URL` env var points to a CDN domain fronting the S3 bucket. See `README.md` for CDN removal steps.
- **Page layout components:** `SectionContainer` (full-width wrapper), `ContentContainer` (max-width), `Grid` (two-column)
- **Site name** configured via `NEXT_PUBLIC_SITE_NAME` env var — used in titles, admin panel, schema markup

## Environment Variables

See `.env.example` for all required variables with descriptions. Key groups:
- **Database:** `MONGODB_URI`, `PAYLOAD_SECRET`
- **Site:** `NEXT_PUBLIC_SITE_URL`, `NEXT_PUBLIC_SITE_NAME`
- **Media:** `S3_ENDPOINT`, `S3_REGION`, `S3_BUCKET_NAME`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `MEDIA_URL` (CDN domain — see README for removal steps)
- **Email:** `RESEND_API_KEY`, `DEFAULT_CONTACT_EMAIL`, `DEFAULT_FROM_EMAIL`, `DEFAULT_FROM_NAME`

## Setup

1. `cp .env.example .env` and fill in values
2. `pnpm install`
3. `pnpm seed` to populate demo content
4. `pnpm dev`

---
> Source: [nick-vogel/midlomark](https://github.com/nick-vogel/midlomark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
