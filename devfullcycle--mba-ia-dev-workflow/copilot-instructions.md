## error-handler

> Use when writing or modifying the global Express error handling middleware. Covers error-to-HTTP mapping pattern and consistent response shape.


# Error Handler Pattern

The `errorHandler.ts` is the single global Express error middleware. It catches all errors forwarded via `next(error)` from controllers and converts them into consistent JSON responses.

## Rules

1. **Standard Express error middleware signature**: `(error, req, res, next)` — all four parameters are required
2. **Map custom error classes to HTTP status codes**: each custom error (e.g., `NotFoundError`, `ConflictError`, `ValidationError`) maps to a specific status code (404, 409, 422, etc.)
3. **Unknown errors return 500**: if the error is not a known custom type, return `500 Internal Server Error` with a generic message — never expose internal details (stack traces, DB errors) to the client
4. **Consistent response shape**: every error response follows the same JSON structure:
   ```json
   {
     "error": {
       "code": "ERROR_CODE",
       "message": "Human-readable message"
     }
   }
   ```
5. **Log before responding**: log the full error (including stack trace) for observability before sending the response — the client gets a safe message, the logs get the full detail
6. **No business logic**: the error handler only maps errors to responses — it never retries, recovers, or makes decisions

---
> Source: [devfullcycle/mba-ia-dev-workflow](https://github.com/devfullcycle/mba-ia-dev-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
