## smarttracking

> Knowledge graph at `graphify-out/` with god nodes and community structure. When the user types `/graphify`, invoke `skill("graphify")` first.

## graphify

Knowledge graph at `graphify-out/` with god nodes and community structure. When the user types `/graphify`, invoke `skill("graphify")` first.

- Always read `graphify-out/GRAPH_REPORT.md` before searching source files
- If `graphify-out/wiki/index.md` exists, use it instead of raw files
- For cross-module questions, prefer `graphify query "<q>"` over grep
- After modifying code, run `graphify update .` (AST-only, no API cost)

**Graph may be stale** — built from commit `67d5e60e`; compare `git rev-parse HEAD` to check.

## Project structure

Monorepo with three directories deployed to Vercel:

| Dir | Tech | Port | Entry |
|-----|------|------|-------|
| `backend/` | NestJS 11 | 3001 | `src/main.ts` |
| `frontend/` | Next.js 16 | 3000 | App Router |
| `payroll-portal/` | Next.js (standalone) | — | App Router |

Root `package.json` only has Vercel analytics deps. All dev happens in subdirectories.

Also notable: `brain/` (unspecified utility code), `scratch/` (gitignored scratch).

## Local dev

```powershell
# Backend (separate terminal)
cd backend; npm run start:dev

# Frontend (separate terminal)
cd frontend; npm run dev
```

Frontend proxies API to `NEXT_PUBLIC_API_URL` (defaults to `http://localhost:3001`).

## Database

PostgreSQL via Supabase, accessed through Prisma with `multiSchema` (`auth` + `public`).

**Two Prisma schemas** — `backend/prisma/schema.prisma` and `frontend/prisma/schema.prisma`. They should reflect the same database but are **not identical**: the frontend schema is a subset (missing `Fabric`, `Tailor`, `FabricTransaction`, `TailoringRequest`, `PayrollRequest`, `Company`, some `Document`/`StorageBatch`/`User` fields). Keep the superset (backend) as source of truth.

Required env vars: `DATABASE_URL`, `DIRECT_URL`, `JWT_SECRET`, `SUPABASE_URL`, `SUPABASE_KEY`.

Backend `postinstall` runs `prisma generate`; Frontend `build` also runs `prisma generate`.

## Auth

Custom JWT (not Supabase Auth client-side). Token in `localStorage`, sent as `Authorization: Bearer <token>` via Axios interceptor (`frontend/src/lib/api.ts`). Backend guards use `@nestjs/jwt`.

## Backend quirks

- **CORS** handled in a manual middleware at `backend/src/main.ts`, not NestJS `enableCors()`
- **Body limit** 50mb for base64 evidence images (`backend/src/main.ts`)
- **Vercel dual-mode** — if `process.env.VERCEL` is set, exports a serverless handler instead of listening
- **ESLint flat config** (`eslint.config.mjs`) — `@typescript-eslint/no-explicit-any: off`, Prettier with `endOfLine: "auto"`
- **Global CacheModule** with 60s TTL — API responses may be cached for 1 minute
- **TrafficInterceptor** logs every request to `TrafficLog` table (global `APP_INTERCEPTOR`)
- **`vercel-build` script**: `prisma generate && nest build`

## Frontend quirks

- **See also `frontend/AGENTS.md`** — warns about Next.js 16 breaking changes
- **Tailwind CSS v4** with `@tailwindcss/postcss` (no classic config file)
- **ESLint flat config** (`eslint.config.mjs`) — `eslint-config-next` core-web-vitals + typescript
- **`@/*`** maps to `./src/*`
- Server actions have **10mb** body limit (`next.config.ts`)
- Uses `sonner` (toasts), `framer-motion` (animations), `lucide-react` (icons)

## Testing

- Backend unit: `cd backend; npm test` (Jest, `*.spec.ts` in `src/`)
- Backend E2E: `cd backend; npm run test:e2e` (Jest, `test/*.e2e-spec.ts`)
- No frontend tests found

## Deployment

- Both `backend/` and `frontend/` have `.vercel/` dirs and `vercel.json` / `next.config.ts`
- Backend CORS origin pinned to `https://smarttracking-frontend.vercel.app` in `vercel.json`
- Backend `vercel-build` hook: `prisma generate && nest build`
- No CI/CD workflows found (`.github/workflows/` absent)

## Style

- Backend: Prettier + ESLint with `endOfLine: "auto"` (Windows compat)
- No code comments convention — avoid adding comments
- `.xlsx` / `.xlsb` files are gitignored (Excel exports not committed)

---
> Source: [edm1922/Smarttracking](https://github.com/edm1922/Smarttracking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
