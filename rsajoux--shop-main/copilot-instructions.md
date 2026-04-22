## shop-main

> Backend architecture and critical rules


# Backend Architecture & Critical Rules

Layered architecture:
Handler -> Validation -> Service -> DB/External

**Handlers (API routes or Route Handlers):**
- Authenticate user
- Validate input with Zod
- Call service methods
- Log with Pino
- Return HTTP responses
- NO business logic, NO direct DB queries

**Services (src/services/):**
- All business logic
- Data transformations
- Database operations
- External API calls
- Return typed data

**Critical rules:**
- NO console.log/warn/error -> Use Pino logger only (createApiLogger or createServiceLogger)
- All inputs (body/query) validated with Zod schemas (src/lib/validation/schemas.ts)
- Use validateRequest helper in handlers
- Separate function per HTTP method in handlers
- Consistent responses: { ok: true, data } or { error: string }
- Proper HTTP status codes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rsajoux) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
