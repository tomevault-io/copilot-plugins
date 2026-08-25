## apen-boas-vindas

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server at http://localhost:3000
npm run build    # Build for production
npm run lint     # Run ESLint
```

TypeScript and ESLint errors are intentionally ignored during build (`ignoreBuildErrors: true`, `ignoreDuringBuilds: true` in `next.config.js`) — check locally before deploying.

## Environment Variables

Required in `.env.local` (see `.env.example`):

| Variable | Used by |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Browser + server |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Browser client (`lib/supabase.ts`) |
| `SUPABASE_SERVICE_ROLE_KEY` | Server-only admin client (`lib/supabase-admin.ts`) |
| `ADMIN_PASSWORD` | `/api/admin/auth` — defaults to `'0197'` if unset |
| `NEXT_PUBLIC_BASE_URL` | `/api/pdf/[id]` redirect |

The Supabase free plan **pauses projects after 7 days of inactivity**. If the app returns 500 errors, check if the project is paused at supabase.com.

## Architecture

### Data model

Two tables in Supabase:

- **`cartas`** — one row per new employee. Contains fixed messages from Saulo Godoy, Túlio Cavalcanti, and Mayra Luna, plus the new employee's photo. Has a `slug` column (unique, nullable) for pretty URLs.
- **`contribuicoes`** — many rows per carta. Each row is a message submitted via the collaborative link. `pagina: 1` = Åpen team (shown in the main letter), `pagina: 2` = family (shown in a separate section with photos).

Two Supabase Storage buckets (both public): `fotos_colaboradores` and `fotos_contribuicoes`.

### Two Supabase clients

- `lib/supabase.ts` — anon key, for browser use
- `lib/supabase-admin.ts` — service role key, lazy-initialized via Proxy, used in all API routes. Never import this in client components.

### URL routing

- `/` — create a new carta (admin form)
- `/carta/[id]` — collaborative link sent to team/family. `[id]` can be a UUID (legacy) or a slug (new cartas). The API route `/api/cartas/[id]` detects which via regex and queries accordingly.
- `/admin/[id]` — password-protected admin panel per carta. Manage contributions, move between pages 1/2, generate PDF.
- `/preview/[id]` — print-ready layout. Contains all CSS inline via `<style dangerouslySetInnerHTML>` in `PreviewContent.tsx`. Use Ctrl+P → Save as PDF with **Margins: None** and **Background graphics** checked.
- `/api/pdf/[id]` — redirects to `/preview/[id]` (Puppeteer was removed; browser print is used instead).

### PDF generation

All print CSS lives in the `PRINT_CSS` constant in `app/preview/PreviewContent.tsx`. Key rules:
- `@page { size: A4; margin: 0 }` — no page margins
- `thead { display: table-header-group }` — header repeats on every page
- Footer (`div.footer-inline`) is an inline block at the end of content, **not** a `tfoot`, to avoid Chrome reserving footer space on every page (caused 100+ page PDFs)
- `print-color-adjust: exact` required for background colors to print

### Fixed team members

Saulo Godoy's message, Túlio Cavalcanti's message, and Mayra Luna's details are **hardcoded** in `app/preview/PreviewContent.tsx` and `lib/pdf-template.ts`. Their photos are served from `public/images/`.

### Slug generation

New cartas get a slug from `nome_colaborador` (lowercased, diacritics removed, spaces → hyphens). Collision handling appends `-2`, `-3`, etc. The `slug` column must exist in Supabase — run `ALTER TABLE cartas ADD COLUMN IF NOT EXISTS slug text unique;` if setting up a new environment.

---
> Source: [mayralunanobre-sudo/apen-boas-vindas](https://github.com/mayralunanobre-sudo/apen-boas-vindas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
