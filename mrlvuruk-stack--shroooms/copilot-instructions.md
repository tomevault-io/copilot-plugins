## shroooms

> Always adhere to these guidelines for all code edits, creations, and commits.

# Cursor Rules for SHROOOMS

## Security & Implementation Rules

Always adhere to these guidelines for all code edits, creations, and commits.

### 1. Secrets and Environment Variables
- Never expose secrets or API keys in frontend code.
- Store all config in `.env` and keep `.env` files in `.gitignore`.
- Use `process.env.VAR_NAME` client-side only if prefixed with `NEXT_PUBLIC_` or `VITE_` (and make sure it is not a private key).

### 2. Rate Limiting
- Apply rate limiting on all public API endpoints (express-rate-limit).
- Limit auth endpoints to 5 requests per 15 minutes.

### 3. Input Validation
- Validate and sanitize all user input on the server side using libraries like Zod or Joi.
- Use parameterized SQL/NoSQL queries to prevent injection attacks.

### 4. Auth & Authz
- Use bcrypt or argon2 for password storage.
- Store refresh tokens in httpOnly cookies.
- Check user permissions and resource ownership on every backend query.

### 5. CORS Configurations
- Never use wildcard `*` CORS in production. Specify exact allowed origins.

### 6. Security Headers
- Use `helmet` in Node/Express to set secure HTTP headers (CSP, X-Frame-Options, HSTS).

### 7. File Uploads
- Validate MIME types, rename files to UUIDs, and store outside the web root.

### 8. Errors & Logging
- Return generic error messages to users. Log verbose stack traces only on the server.

### 9. Dependencies
- Audit packages frequently (`npm audit`) and pin versions.

### 10. XSS Prevention
- Never render user-generated content as raw HTML without sanitizing using DOMPurify.
- Avoid using React's `dangerouslySetInnerHTML`.

### 11. AI & LLM Rules
- Treat all LLM inputs and outputs as untrusted.
- Limit max tokens, route keys through the server only, and sanitize inputs to prevent prompt injection.

---
> Source: [mrlvuruk-stack/shroooms](https://github.com/mrlvuruk-stack/shroooms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
