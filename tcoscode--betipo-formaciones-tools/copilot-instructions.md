## betipo-formaciones-tools

> - Use `npm` in this repo; `package-lock.json` is the lockfile.

# AGENTS.md

## Commands

- Use `npm` in this repo; `package-lock.json` is the lockfile.
- Use Node `22.12.0` from `.nvmrc`.
- Start dev server: `npm run dev` (`astro dev`, default port 4321).
- Full local verification: `npm run verify` runs `format:check -> lint -> test -> check -> build`.
- Focused checks: `npm run test -- src/lib/env.test.ts`, `npm run lint`, `npm run check`, `npm run build`.
- Formatting is Prettier with Astro and Tailwind plugins: `npm run format` or `npm run format:check`.

## App Shape

- This is an Astro SSR app (`output: "server"`) deployed through the Netlify adapter with edge middleware enabled; Netlify builds with `npm run build` and publishes `dist`.
- The page entrypoint is `src/pages/index.astro`, which hydrates the React dashboard via `DashboardTabs client:load`.
- API routes live under `src/pages/api/**`; they are not prerendered and query Postgres directly through `src/lib/db.ts`.
- The dashboard manages two database environments via `?env=dev|prod`; omitted `env` defaults to `dev`, and invalid values return 400.

## Data And Env

- Database credentials are read with `astro:env/server` at module import time in `src/lib/db.ts`; builds/API routes need `DB_HOST`, `DB_USERNAME`, `DB_PASSWORD`, `DB_DATABASE`, and `DB_DATABASE_PROD`; `DB_PORT` defaults to `5432`.
- `prod` in the UI/API targets the production database (`DB_DATABASE_PROD`); preserve the existing confirmation flow (`confirmProductionEnv` in `src/utils/production.ts`, SweetAlert2) before changing production-affecting behavior.
- Date inputs are shown in `Europe/Madrid`, converted to UTC before API submission, and DB timestamp text is normalized by `dbDateToUtc`; keep related tests in `src/utils/dates.test.ts` updated when touching this flow.

## Testing Notes

- Tests are Vitest unit tests colocated in `src/**/*.test.ts`; there is no Vitest config file.
- Current tests cover env parsing, input validation, and date conversion. Add focused unit coverage there when changing those utilities or schemas.

## API Route Conventions

- Each API route under `src/pages/api/**` declares `export const prerender = false` explicitly, even though `output: "server"` is set globally.
- Standard flow: `parseEnv(url)` → return 400 on `null`, `getDb(env)` for the connection, validate body with schemas from `src/lib/validation.ts`, respond via `json()` from `src/lib/http.ts`, and normalize DB timestamp text with `dbDateToUtc` before returning dates. See `src/pages/api/formaciones/index.ts` as the reference implementation.

## Config Notes

- TypeScript extends `astro/tsconfigs/strict`; React JSX is configured in `tsconfig.json`.
- ESLint uses flat config for JS, TypeScript, Astro, React, and React Hooks; `react-hooks/set-state-in-effect` is intentionally disabled.
- Repo-local `opencode.json` enables the Astro docs MCP; prefer it when checking Astro-specific behavior.

---
> Source: [tcosCode/betipo-formaciones-tools](https://github.com/tcosCode/betipo-formaciones-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
