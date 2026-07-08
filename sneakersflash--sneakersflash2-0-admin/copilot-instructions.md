## sneakersflash2-0-admin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Dev Setup

This frontend requires the `sneakersflash-backend` running locally first. The default `NEXT_PUBLIC_API_URL` points to `http://localhost:3000/api/v1/`. Start the backend before running `npm run dev`.

Required env vars (`.env.local`):
- `NEXT_PUBLIC_API_URL` — backend API base URL
- `NEXT_PUBLIC_APP_NAME` — app branding
- `JWT_SECRET` — used for cookie verification in middleware

## Commands

```bash
npm run dev          # start dev server
npm run build        # production build (outputs standalone)
npm run lint         # ESLint (Next.js flat config v9)
npm run type-check   # tsc --noEmit
```

## Code Style

- Prefer `type` over `interface` for all TypeScript type definitions.
- Use named exports; avoid default exports except for Next.js page files.
- Path alias `@/*` resolves to `src/*`.
- TailwindCSS v4 — there is no `tailwind.config.js`. Config lives inline in `src/app/globals.css` using `@theme`. Do not create a config file.
- shadcn/ui components use new-york style with slate base color. Add new primitives via `npx shadcn add <component>`.

## Module Architecture

Each dashboard feature follows this structure:

```
src/
  app/(dashboard)/dashboard/<module>/    # Next.js page route
  components/module/<module>/           # Feature UI components
  services/<module>.service.ts          # All API calls for the module
  types/<module>.types.ts               # TypeScript types
  lib/validators/<module>.validator.ts  # Zod schemas (optional)
```

When adding a new module, follow this pattern. Service functions use the Axios instance from `src/lib/api.ts`.

## Authentication

Cookie-based JWT stored in `sf_access_token`. The middleware at `src/middleware.ts` protects `/dashboard/*` routes and redirects unauthenticated users to `/login`. Do not use localStorage for tokens.

## Git Conventions

Branch naming: `feature/*`, `fix/*`, `chore/*`

---
> Source: [SneakersFlash/SneakersFlash2.0-Admin](https://github.com/SneakersFlash/SneakersFlash2.0-Admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
