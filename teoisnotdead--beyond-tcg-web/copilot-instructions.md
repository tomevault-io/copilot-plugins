## beyond-tcg-web

> - Use npm; `package-lock.json` is the only lockfile. Avoid adding pnpm/yarn lockfiles.

# AGENTS.md

## Commands
- Use npm; `package-lock.json` is the only lockfile. Avoid adding pnpm/yarn lockfiles.
- `npm run dev` starts Next dev on `http://localhost:3000`.
- `npm run build` is the production/TypeScript verification path.
- `npm run lint` runs ESLint 9 with `eslint-config-next/core-web-vitals` and TypeScript rules.
- There is no test runner, formatter, or `typecheck` script in this repo. If you need a focused type check, run `npx tsc --noEmit`.

## App Wiring
- This is a single Next.js 16 App Router app, not a monorepo. Main route groups are `app/(marketplace)`, `app/(auth)`, `app/auth/callback`, and `app/(dashboard)`.
- `app/layout.tsx` wraps all routes with `app/providers.tsx`; keep React Query, `next-themes`, and the axios 401 bridge there when changing global providers.
- Dashboard protection is client-side in `app/(dashboard)/layout.tsx` via `useAuth()`, a loading state, and `router.replace('/login')`. Do not move dashboard pages outside that group unless you add equivalent protection.
- Hooks used by app code live under `lib/hooks/*`; the `components.json` shadcn alias for `hooks` points to `@/hooks`, but that directory does not currently exist.

## API And Auth
- Use `apiClient` from `lib/api/client.ts` for backend calls. It prefixes `${NEXT_PUBLIC_API_URL || 'http://localhost:3001'}/api` and injects the public metadata headers from env.
- Auth state lives in Zustand at `lib/store/auth-store.ts`. Persisted keys are `beyond-tcg-token`, `beyond-tcg-user`, and session marker `beyond-tcg-session`.
- `apiClient` automatically adds `Authorization` from localStorage unless the request config has `skipAuth`. Keep browser storage access guarded with `typeof window !== 'undefined'`.
- 401 handling is registered both in the store fallback and in `app/providers.tsx`; prefer the provider bridge for app-router navigation instead of hard reloads.
- `useSales` normalizes category/language arrays by sorting and comma-joining them before sending `/sales` params and building the React Query key. Preserve that when adding filters.

## Styling And UI
- Tailwind CSS v4 is configured inline in `app/globals.css` with `@theme`, `@custom-variant dark`, and CSS variables; there is no `tailwind.config.*`.
- shadcn/ui settings are in `components.json`: New York style, RSC enabled, neutral base color, Lucide icons, UI components under `components/ui`.
- `next.config.ts` only allows optimized remote images from `https://res.cloudinary.com`; add new image hosts there before using `next/image` with other domains.

## Environment
- `.env*` files are gitignored. Required public envs are documented in `README.md`; only `NEXT_PUBLIC_API_URL` changes backend routing, the rest populate request headers.

---
> Source: [teoisnotdead/beyond-tcg-web](https://github.com/teoisnotdead/beyond-tcg-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
