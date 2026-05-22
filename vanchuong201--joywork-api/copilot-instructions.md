## backend-api

> JoyWork backend controller, service, auth, and response conventions


# JoyWork Backend API

- Controllers should parse input, call services, and return the standard success envelope `{ data: ... }`; keep business logic in services.
- New success responses should stay in the `{ data: ... }` envelope; do not introduce `success: true` unless you are intentionally preserving a legacy contract.
- Domain failures should throw `AppError` so status codes and machine-readable error codes stay consistent; avoid hand-building `{ error: ... }` in controllers except for special response types like redirects or file flows.
- Protected routes must use the existing auth middleware and read the current actor from `request.user?.userId`.
- Add or update Fastify route schemas for params, body, querystring, and response whenever an endpoint contract changes so Swagger stays accurate.
- Keep route schema and Zod validation aligned; do not update one without the other.
- Prefer existing module naming and file layout conventions over creating custom patterns for one endpoint.
- Do not add test, demo, or temporary debug routes inside production route files.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
