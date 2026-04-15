## afflux

> Afflux is an AI-powered TikTok Shop affiliate marketing SaaS platform. It automates creator discovery, outreach, relationship management, and performance analytics.

# Afflux — Project Rules

## What is this?

Afflux is an AI-powered TikTok Shop affiliate marketing SaaS platform. It automates creator discovery, outreach, relationship management, and performance analytics.

## Tech Stack

- **Frontend**: Next.js 16 (App Router), TypeScript, Tailwind CSS, shadcn/ui
- **Backend**: tRPC (type-safe API), Hono (external API)
- **Database**: PostgreSQL (Neon) + Drizzle ORM
- **Cache/Queue**: Redis + BullMQ
- **AI**: Claude API (Anthropic), OpenAI Embeddings
- **i18n**: next-intl (multi-locale routing, ICU message format)

## Project Structure

```
src/
├── app/[locale]/           # Next.js App Router (locale-prefixed routes)
│   ├── (dashboard)/        # Dashboard layout group
│   ├── (auth)/             # Auth layout group
│   └── layout.tsx          # Locale layout with i18n provider
├── components/
│   ├── ui/                 # shadcn/ui components (DO NOT manually edit)
��   ├── layout/             # App shell, sidebar, header
│   ├── creators/           # Creator search, profile, list components
│   ├── outreach/           # Outreach campaign components
│   ├── crm/                # CRM pipeline, contact components
│   └── providers/          # Context providers (theme, auth, etc.)
├── lib/
│   ├─��� db/                 # Drizzle schema and database client
│   ├── trpc/               # tRPC router and context setup
│   ├── i18n/               # next-intl config, routing, formatters
│   ├── ai/                 # AI service layer (matching, generation)
│   ├── auth/               # Authentication logic
│   └── queue/              # BullMQ job definitions
├── server/
│   ├── routers/            # tRPC routers (creator, outreach, crm, etc.)
│   └── services/           # Business logic services
├── hooks/                  # Custom React hooks
└── types/                  # Shared TypeScript types
locales/
├── en/                     # English translations
└── ko/                     # Korean translations
```

## Key Conventions

- **i18n-First**: All user-facing text MUST use translation keys via `useTranslations()`. Never hardcode UI strings.
- **Multi-tenant**: All database queries MUST include `tenantId` filter. Use `protectedProcedure` from tRPC.
- **Server Components by default**: Only add `'use client'` where interactivity is required.
- **Dark mode default**: Use shadcn theme tokens (`bg-background`, `text-foreground`, etc.), never hardcoded colors.
- **Drizzle migrations**: Run `pnpm db:generate` after schema changes, then `pnpm db:push`.

## Commands

```bash
pnpm dev            # Start dev server
pnpm build          # Production build
pnpm db:generate    # Generate Drizzle migrations
pnpm db:push        # Push schema to database
pnpm db:studio      # Open Drizzle Studio
pnpm lint           # Run ESLint
pnpm test           # Run Vitest
```

## Don't

- Don't use `@vercel/postgres` or `@vercel/kv` — use Neon + Drizzle and Redis directly
- Don't bypass multi-tenancy — every data query must be tenant-scoped
- Don't hardcode English strings in components — always use translation keys
- Don't put secrets in `NEXT_PUBLIC_` env vars

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onlyhyde) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
