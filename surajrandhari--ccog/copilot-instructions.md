## ccog

> Full-stack church website and CMS built with Next.js 15 (App Router), TypeScript, Tailwind CSS, shadcn/ui, and MongoDB Atlas. Deployed on Vercel.

# Calvary Church of God — Digital Platform

## Project Overview
Full-stack church website and CMS built with Next.js 15 (App Router), TypeScript, Tailwind CSS, shadcn/ui, and MongoDB Atlas. Deployed on Vercel.

Three consumers share one codebase:
- Public website (`app/(site)/`) — SSG + ISR
- CMS admin dashboard (`app/admin/`) — SSR, auth-protected
- REST API (`app/api/v1/`) — for future Flutter/React Native mobile apps

## Architecture Docs
Read these before starting any task. Reference them with @-mentions as needed.

- @docs/product-vision.md — Design philosophy, feature list, product goals
- @docs/system-architecture.md — Full technical blueprint, API design, data flow, deployment
- @docs/database-design.md — All MongoDB schemas, indexes, relationships
- @docs/cms-system.md — CMS dashboard structure, roles, workflows, auth

## Tech Stack
- **Framework**: Next.js 15, App Router, TypeScript (strict mode)
- **Styling**: Tailwind CSS + shadcn/ui + Radix UI primitives
- **Animations**: Framer Motion (micro-interactions only, never impact CWV)
- **Database**: MongoDB Atlas via Mongoose
- **Auth**: JWT in httpOnly cookie (web) + Bearer token (mobile API)
- **Media**: Cloudinary (all uploads go through server, never direct browser upload)
- **Email**: Resend
- **Rate limiting**: Upstash Redis
- **Deployment**: Vercel

## Commands
```bash
npm run dev          # Start dev server
npm run build        # Production build
npm run lint         # ESLint check
npm run typecheck    # tsc --noEmit
npx tsx scripts/seed.ts  # Seed database with sample data
```

## Code Rules
- Use Server Actions for all data mutations — no separate API routes for CMS writes
- All public REST endpoints live under `app/api/v1/`
- Every Mongoose model lives in `src/models/` — one file per collection
- Zod schemas for all input validation — live in `src/lib/validations/`
- Never hardcode secrets — use `process.env.*` only
- Use `next/image` for all images — Cloudinary as source
- Use `next/font` for all fonts
- `revalidatePath()` or `revalidateTag()` must be called in every Server Action that mutates published content
- All admin routes are protected by middleware — never trust client-side auth checks alone
- DOMPurify sanitisation on all rich text HTML before rendering with `dangerouslySetInnerHTML`

## Design Rules
- Apple-level minimalism: white-first, wide spacing, calm typography
- Framer Motion only for micro-interactions — no GSAP unless absolutely required for complex sequences
- All motion must respect `prefers-reduced-motion`
- shadcn/ui components as the base — extend, don't replace
- Lighthouse 95+ target — no lazy loading violations, no CLS issues

## File Structure
```
src/
├── app/
│   ├── (site)/          # Public website pages
│   ├── admin/           # CMS dashboard
│   └── api/v1/          # REST API
├── components/
│   ├── admin/           # CMS-specific components
│   ├── site/            # Public site components
│   └── ui/              # shadcn/ui components
├── lib/
│   ├── auth.ts          # JWT helpers
│   ├── mongodb.ts       # Connection singleton
│   ├── cloudinary.ts    # Upload helpers
│   └── validations/     # Zod schemas
└── models/              # Mongoose models
```

## Environment Variables Needed
```
MONGODB_URI
JWT_SECRET
CLOUDINARY_CLOUD_NAME
CLOUDINARY_API_KEY
CLOUDINARY_API_SECRET
RESEND_API_KEY
UPSTASH_REDIS_REST_URL
UPSTASH_REDIS_REST_TOKEN
NEXT_PUBLIC_SITE_URL
```

---
> Source: [SurajRandhari/ccog](https://github.com/SurajRandhari/ccog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
