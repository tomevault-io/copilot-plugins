## eachlabs-ai-logo-maker-saas

> This is a **Turborepo monorepo** with Bun workspaces:

# Repository Guidelines

## Project Structure & Module Organization

This is a **Turborepo monorepo** with Bun workspaces:

```
apps/
├── web/                 # Next.js 15 frontend (port 3000)
│   ├── app/             # App Router pages, layouts, styles
│   ├── components/      # UI + landing sections (logo-maker.tsx, landing/*)
│   ├── hooks/           # Custom React hooks
│   ├── lib/             # Shared helpers (utils.ts with cn function)
│   └── public/          # Static assets
│
└── api/                 # Hono + Bun API server (port 3002)
    ├── src/
    │   ├── auth.ts      # Better Auth server instance (drizzle adapter, anonymous plugin)
    │   ├── db/          # Drizzle schema, connection, migrations
    │   ├── routes/      # Hono routes (predictions.ts)
    │   └── index.ts     # Hono entrypoint
    └── src/db/migrations/ # Generated migrations (Drizzle + Better Auth)

docs/                    # Documentation (API registry, PRD, platform notes)
```

## Build, Test, and Development Commands

```bash
# Monorepo commands (from root)
bun dev                        # Start both web and API servers
bun run build                  # Build all apps
bun lint                       # ESLint checks on all apps
bun start                      # Start production servers

# Filter to specific app
bun run dev -- --filter=web    # Only web app
bun run dev -- --filter=api    # Only API server

# Database (Drizzle ORM, runs against api app)
bun run db:generate            # Generate migrations from schema changes
bun run db:migrate             # Run pending migrations
bun run db:push                # Push schema directly (dev only)
bun run db:studio              # Open Drizzle Studio GUI
```

## Coding Style & Naming Conventions
- TypeScript-first; App Router for web, Hono for API. Prefer functional components with hooks.
- Tailwind CSS v4 utility classes; avoid inline styles unless needed for animations.
- Use `cn` helper for class merging. Keep copy in English.
- Linting: ESLint 9. Run `bun lint` before pushing.
- Filenames: kebab-case for files, PascalCase for components, camelCase for variables/functions.
- Client data fetching/polling (predictions, auth, etc.) should go through TanStack Query. Shared QueryClient lives in `apps/web/components/providers/query-provider.tsx` and is mounted via `AppProvider` in `app/layout.tsx`.

## Testing Guidelines
- No formal test suite yet. Add tests alongside features when feasible.
- For API logic, prefer lightweight integration tests and unit tests for helpers.
- Name test files `*.test.ts` / `*.test.tsx`; colocate near source or under `__tests__/`.

## Commit & Pull Request Guidelines
- Commit messages: short imperative summary (e.g., "Rebrand app to LogoLoco", "Update README for bun workflow").
- Pull requests: include summary of changes, relevant screenshots for UI tweaks, and note any lint/test runs. Link issues when applicable.

## Security & Configuration Tips
- Required envs for API: `DATABASE_URL`, `EACHLABS_API_KEY`, `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL`; set `DATABASE_SSL=true` in production. Use `SIGNUP_BONUS_CREDITS` to set the initial free credit grant and `ADMIN_EMAILS` for `/api/admin/*` access.
- Required envs for Web: `NEXT_PUBLIC_API_BASE_URL` (default: `http://localhost:3002`).
- CORS: `ALLOWED_ORIGINS` comma-separated list for the API.
- History retention: `GENERATION_RETENTION_DAYS` (default 365) applies to `/api/predictions` history listing.
- Polar checkout: `POLAR_ACCESS_TOKEN` (required), `POLAR_SERVER` (`sandbox`|`production`, default `sandbox`), `POLAR_CHECKOUT_SUCCESS_URL` (include `{CHECKOUT_ID}` placeholder), `POLAR_CHECKOUT_RETURN_URL` (optional back link).
- Do not commit secrets. `.env` / `.env.local` files are ignored.

## Architecture Overview
- Auth: Better Auth hosted in API, handler at `/api/auth/*`; Next.js uses auth client pointing at API base URL.
- Logo flow: Client submits form → `POST /api/predictions` persists request + calls Eachlabs (charges 1 credit per requested output, max 4) → client polls `GET /api/predictions/{id}` (auth + owner-only) until status is `succeeded` (implemented with TanStack Query polling).
- History: `GET /api/predictions` is auth-only, paginated (limit/offset), filtered to the requesting user, and retention-limited.
- DB table: `logo_generations` with status/images/provider IDs; indexes on `created_at`, `status`, `provider_prediction_id`.
- Model mapping: `nano-banana` → `nano-banana`, `seedream-v4` → `seedream-v4-text-to-image`, `reve-text` → `reve-text-to-image`.

---
> Source: [altudev/eachlabs-ai-logo-maker-saas](https://github.com/altudev/eachlabs-ai-logo-maker-saas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
