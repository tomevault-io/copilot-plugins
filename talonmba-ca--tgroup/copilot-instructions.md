## tgroup

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # Start dev server on port 3004 with Turbopack
pnpm build        # Production build
pnpm lint         # Run ESLint
```

## Architecture

This is a Next.js 15 App Router project for Talonmba Formation, a professional training company. The site supports three languages (French, English, Haitian Creole).

### Internationalization

- Uses `next-international` for i18n with `[locale]` dynamic route segment
- Supported locales: `fr` (default), `en`, `ht`
- Locale files in `locales/` - use `client.ts` hooks in client components, `server.ts` functions in server components
- Middleware handles locale detection and URL rewriting (`urlMappingStrategy: 'rewriteDefault'` hides `/fr` prefix)

### Key Directories

- `app/[locale]/` - Page routes (contact, faq, about, blog)
- `actions/` - Server Actions (e.g., `handleContactSubmit` for form submissions)
- `components/` - UI organized by: `ui/` (shadcn primitives), `home/`, `layout/`, `icons/`, `errors/`, `global/`
- `validations/` - Zod schemas (configured with French locale)
- `interfaces/` - TypeScript interfaces for API entities (Contact, Blog, Formation, User)
- `lib/utils.ts` - Utilities including `cn()` for classnames, `http` axios instance, `slugify`, `formatAmount`

### External API

The app communicates with an external API via:
- Server-side: `fetch()` with `BASE_URL_API` env var
- Client-side: `http` axios instance from `lib/utils.ts`
- Authentication via custom token headers (`GUEST_TOKEN_NAME`/`GUEST_TOKEN_VALUE`)

### Path Alias

`@/*` maps to project root (e.g., `@/components`, `@/lib/utils`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/talonmba-ca) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
