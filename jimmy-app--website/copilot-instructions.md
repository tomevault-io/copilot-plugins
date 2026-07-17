## website

> Next.js 16 · TypeScript strict · Tailwind CSS v4 · Sanity v5 · next-intl v4 (en/fr/es)

# Jimmy Marketing — Claude Code Guide

## Stack
Next.js 16 · TypeScript strict · Tailwind CSS v4 · Sanity v5 · next-intl v4 (en/fr/es)

## Available Skills
Use these slash commands for guidance:
- `/nextjs` — Next.js 16 App Router best practices
- `/next-forge` — architecture patterns
- `/next-cache-components` — `use cache` / PPR caching strategy

## Key Paths
- `src/app/[locale]/` — public pages (always with locale prefix)
- `src/app/studio/` — Sanity Studio (no locale, excluded from middleware.ts)
- `src/components/` — components (structure per PLAN.md)
- `sanity/schemas/` — one schema = one file
- `sanity/queries/` — all GROQ queries here
- `messages/` — UI translations (en / fr / es)

## Rules

**TypeScript** — strict, no `any`, no `@ts-ignore`

**Tailwind v4** — no tailwind.config.js.
Tokens in `src/styles/globals.css` via `@theme {}`.
Conditional classes via `cn()` from `@/lib/utils`.

**i18n** — all UI text via `getTranslations()` (server) or `useTranslations()` (client).
New keys go into ALL 3 messages/ files immediately.
Studio `/studio` excluded from middleware.ts matcher.

**Components** — Server Components by default.
`use client` only at the leaf where interactivity is required.

**Caching** — use `'use cache'` directive with `cacheTag(\`{type}-{locale}\`)`.
Sanity content cached with `cacheLife('hours')`.
Revalidate via POST `/api/revalidate?secret=...`.

**Sanity** — after changing schemas: `npm run typegen`.
GROQ queries only in `sanity/queries/index.ts`.

**Icons** — only `lucide-react`
**Animations** — only `framer-motion` (only in Client Components)

## Commands
```bash
npm run dev        # Next.js + Turbopack
npm run build      # Production build
npm run type-check # TypeScript without build
npm run typegen    # Regenerate Sanity types
```

## Studio
Dev: http://localhost:3000/studio

## Adding New Content Type
1. Create schema in `sanity/schemas/documents/` or `sanity/schemas/objects/`
2. Register in `sanity/schemas/index.ts`
3. Add to `localizedSchemaTypes` in `sanity/sanity.config.ts` (if localized)
4. Run `npm run typegen`
5. Add GROQ query in `sanity/queries/index.ts`
6. Add translations in all 3 messages/ files

---
> Source: [Jimmy-App/website](https://github.com/Jimmy-App/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
