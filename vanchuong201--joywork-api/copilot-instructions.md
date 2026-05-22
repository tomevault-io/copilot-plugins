## security

> JoyWork backend security, input validation, and secret handling


# JoyWork Backend Security

- Never commit or expose secrets from `.env*`, JWT secrets, S3 credentials, SES credentials, cookies, or signed URLs.
- Treat request bodies, query params, HTML, uploads, and third-party payloads as untrusted input and validate them before use.
- Enforce authorization in backend routes and services, especially for company membership, ownership, and privileged downloads or writes.
- Preserve upload restrictions for MIME type, file size, key prefix, and filename sanitization; avoid broad object deletion behavior.
- Keep auth cookies and token-related responses aligned with the current security model; do not weaken `httpOnly`, env-based `secure`, or `sameSite` defaults casually.
- Avoid leaking account existence or internal authorization details in user-facing error messages when a generic success or generic denial is safer.
- Do not leak raw stack traces, Prisma internals, or sensitive operational details in API responses.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
