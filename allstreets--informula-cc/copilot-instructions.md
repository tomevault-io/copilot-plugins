## informula-cc

> Build **INFOrmula** into a runnable beta scaffold without losing product identity.

# AGENTS.md

## Repo purpose
Build **INFOrmula** into a runnable beta scaffold without losing product identity.

This repo now has a working install/typecheck/build path and a mock-safe worker, so future agents should extend it rather than replacing it wholesale.

## Read first
- `README.md`
- `HANDOFF_PROMPT.md`
- `SETUP.md`
- `ENVIRONMENT.md`
- `MVP_SCOPE.md`
- `ACCEPTANCE_CRITERIA.md`
- `TESTING.md`
- `supabase/schema.sql`

## Hard product decisions already made
- Product name is **INFOrmula**
- Auth is **magic-link email** for v1
- Google is the first planned social provider after magic-link, but not part of v1 default auth
- Drivers, teams, seasons, tracks, practice, qualifying, sprint, and race are in scope
- New users start with **$1,000** in beta balance
- Product remains simulation-only
- Market design should preserve a path toward **hybrid CLOB + AMM**
- Admin automation can assist settlement, but high-risk cases keep manual control
- Data pages should feel luxury editorial + high-tech
- Market pages should feel terminal/trading
- Core visual direction is navy + maroon

## Golden rules
1. Work only in this canonical repo.
2. Preserve mock mode.
3. Keep the repo usable without secrets.
4. Do not introduce real-money mechanics.
5. Keep server-only secrets off the client.
6. Keep persistent worker behavior out of ephemeral web handlers.
7. Keep SQL and query logic explicit.
8. Do not flatten archived reference packages into this repo.

## Repo truth as of this handoff
- `pnpm typecheck` passes
- `pnpm build` passes
- worker can run once in mock mode from built output
- Supabase schema includes wallet/profile trigger plus market RPCs
- Next build currently auto-adjusted `apps/web/tsconfig.json` to include `.next/types/**/*.ts` and `esModuleInterop`

## What to report after changes
- files changed
- commands run
- whether mock mode still works
- whether real Supabase mode changed
- what still depends on secrets or paid services

---
> Source: [AllStreets/INFOrmula-cc-](https://github.com/AllStreets/INFOrmula-cc-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
