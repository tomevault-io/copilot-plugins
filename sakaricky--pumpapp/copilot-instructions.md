## api

> API standards — Zod validation, structured errors, no secrets in logs


# API standards

- **Validate input:** Use Zod (or similar) for request bodies and query params. Parse and validate before use; return 400 with clear messages on failure.

- **Error handling:** Use a shared error type and map to HTTP status codes. Do not expose raw stack traces or internal errors in responses; return structured error payloads (e.g. `{ error: string, code?: string }`).

- **No secrets in logs:** Never log passwords, tokens, API keys, or full env objects. Log only safe identifiers (e.g. request id, user id) and redact sensitive fields.

---
> Source: [SakaRicky/PumpApp](https://github.com/SakaRicky/PumpApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
