## cusown

> Config, env, and constants usage


# Config and constants

## Where to put values
- **User-facing strings**: `config/constants.ts` — `ERROR_MESSAGES`, `SUCCESS_MESSAGES`, `VALIDATION`, etc.
- **Env-driven values**: `config/env.ts` — single `env` object; use `env.booking.expiryHours`, `env.security.signedUrlTtlSeconds`, etc. Add new keys to `env` and document in `env.template`.
- **Rate limits, metrics names**: `config/constants.ts` — e.g. `RATE_LIMIT_BOOKING_WINDOW_MS`, `METRICS_BOOKING_CREATED`.

## Rules
- Do not use `process.env` directly in services, API routes, or lib; use `config/env.ts` or `config/constants.ts`.
- Do not add magic numbers for timeouts, limits, or status strings; add named constants and use them.
- Phase policy files (`config/*.policy.ts`) document locked scope; do not change behaviour that contradicts them without approval.

## Adding new constants
- Add to `constants.ts` with `as const` where appropriate. For env-backed values, add to `env` in `config/env.ts` and to `env.template` with a short comment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Clykur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
