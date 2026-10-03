## api-and-auth

> Auth cookie 7d, middleware, next allowlist, route helpers


# API & auth

- Orval: root `orval.config.ts` + `npm run api:generate` → `src/api/generated/` (gitignored; also via `prebuild`/`predev`).
- Axios: `src/lib/api/client.ts`. Cookie `access_token` → Bearer header.
- `Accept-Language: en|fa` on every request (path → `NEXT_LOCALE` → default `fa`).
- Cookie: Secure + SameSite=Lax + Path=/; **7-day** expiry; frontend sets after login.
- Paths via `routes.*()` only. Login `/{locale}`; private app e.g. `routes.home()`.
- Middleware: no cookie on private routes → `routes.auth.login({ next })` (allowlisted `next` only).
- Authed on login → `routes.home()`. Logout via user store.
- `/me` panel-only → Zustand. 401 → unauthorized (+ login button). 403 → access-denied.

---
> Source: [soheilhasanjani/contently](https://github.com/soheilhasanjani/contently) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
