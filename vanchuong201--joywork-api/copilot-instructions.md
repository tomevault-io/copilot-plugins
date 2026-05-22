## api-contracts

> JoyWork API response contract and compatibility rules


# JoyWork API Contracts

- Keep the primary API contract consistent: success responses use `{ data: ... }` and failures use `{ error: { code, message, details? } }`.
- Do not introduce new top-level response shapes for new endpoints; preserve legacy fields like `success` or `reacted` only when maintaining an existing endpoint contract.
- Keep controller payload field names aligned with what the frontend already reads directly, such as `accessToken`, `user`, `company`, `job`, `posts`, `pagination`, or `url`.
- When an endpoint contract changes, update the route schema, controller response, service DTO shaping, and counterpart frontend consumers in the same task.
- Prefer stable, explicit DTO shaping over leaking raw Prisma records when the frontend depends on a curated response shape.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
