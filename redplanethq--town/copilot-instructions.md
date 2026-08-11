## town

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Workspace layout

pnpm + Turbo monorepo.

```
apps/web/                 ← Next.js app (kaplay game + UI + routes)
packages/types/           ← shared TS types (Plot, Variant, TownState, EventEnvelope)
packages/db/              ← Prisma schema + client (@town/db)
```

Common commands (run from repo root):

- `pnpm dev` — `turbo run dev` → starts the web app
- `pnpm build` — production build
- `pnpm typecheck` — every package
- `pnpm db:migrate` / `db:generate` / `db:studio` — proxy into `@town/db`

`.env` lives at the repo root. `apps/web/.env` is a symlink to it so Next
picks it up; `@town/db` scripts load it via `dotenv-cli`.

## Auth

CORE OAuth2 + PKCE. Backend is Postgres + Prisma; the browser only ever
sees an opaque session cookie (`core-town:sid`). Access + refresh tokens
live in the `Session` table.

- Client entry points: `apps/web/src/game/auth.ts` (`startLogin`, `logout`,
  `refreshSession`, `getSession`).
- Server endpoints: `apps/web/src/app/api/auth/{login,callback,me,logout}/route.ts`.
- CORE wire calls: `apps/web/src/lib/oauth.ts`. Session bookkeeping + refresh:
  `apps/web/src/lib/session.ts`. Prisma client: `@town/db` (re-exported by
  `apps/web/src/lib/db.ts` for existing imports).

Local setup:

```bash
cp .env.example .env
# Fill DATABASE_URL + the CORE_OAUTH_* vars.
# CORE_OAUTH_CLIENT_ID/SECRET come from POST /api/oauth/clients on
# app.getcore.me — see comments in .env.example for the payload.
pnpm install
pnpm db:migrate --name init
pnpm dev
```

To call CORE from a Route Handler, read the session id from the cookie,
resolve a fresh access token via `getAccessTokenForSession(sid)`, then
`fetch(CORE_OAUTH_BASE + '/api/v1/...', { headers: { authorization: 'Bearer ' + token } })`.

---
> Source: [RedPlanetHQ/town](https://github.com/RedPlanetHQ/town) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
