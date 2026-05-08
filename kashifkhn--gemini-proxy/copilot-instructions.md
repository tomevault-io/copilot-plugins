## gemini-proxy

> Guidelines for agentic coding agents working in this repository.

# AGENTS.md

Guidelines for agentic coding agents working in this repository.

---

## Project Overview

`gemini-proxy` is a self-hosted OpenAI-compatible HTTP proxy that translates
OpenAI Chat Completions requests to Google Gemini via the Cloud Code Assist
internal API. Runtime: **Bun**. Framework: **Hono**. Language: **TypeScript**.

---

## Commands

### Run (development)
```sh
bun --watch src/index.ts
```

### Run (production)
```sh
bun src/index.ts
```

### Type-check
```sh
bunx tsc --noEmit
```

### No test suite exists yet
There are no automated tests. Manual testing is done with `test-sdk.ts`:
```sh
bun test-sdk.ts
```

There is no lint config (no ESLint, Biome, Prettier). TypeScript strict mode
is the primary correctness enforcement — always run `bunx tsc --noEmit` after
changes.

---

## Environment

Required `.env` variables (see `.env.example`):
- `PROXY_API_KEY` — static Bearer token clients must send
- `DISABLE_AUTH_ENDPOINTS` — set to `"true"` on the server to make `/auth/login` and `/auth/callback` return 404; auth is done locally and `tokens.json` is copied to the server via SCP
- `PROXY_PORT` — optional, defaults to `3000`
- `TOKEN_STORE_PATH` — optional, defaults to `./tokens.json`

---

## Directory Structure

```
src/
  index.ts          entry point, reads env, starts Bun server
  app.ts            Hono app factory, route registration, global error handler
  constants.ts      all magic values, URLs, model list, config helpers
  types.ts          all TypeScript types and interfaces (no logic)
  middleware/
    auth.ts         requireApiKey middleware
  routes/
    auth.ts         /auth/login, /auth/callback, /auth/status
    chat.ts         POST /v1/chat/completions (Zod validation + dispatch)
    models.ts       GET /v1/models
  gemini/
    request.ts      OpenAI → Gemini request translation
    response.ts     Gemini → OpenAI non-streaming response translation
    stream.ts       Gemini SSE → OpenAI SSE stream translation
  store/
    tokens.ts       read/write/refresh token store (tokens.json)
  oauth/
    exchange.ts     PKCE OAuth2 authorization URL + code exchange
    pkce.ts         PKCE challenge/verifier generation
    project.ts      Cloud Code Assist project resolution + onboarding
    refresh.ts      access token refresh with in-flight deduplication
    userAgent.ts    User-Agent string builder
```

---

## Code Style

### General
- **No comments.** Do not write inline comments, block comments, or JSDoc.
  The only exception is when something is genuinely non-obvious and critically
  important — and even then, keep it to one short line maximum.
- Prefer clarity through naming over explaining with comments.
- Keep functions small and single-purpose.
- No unused variables, no dead code.

### TypeScript
- Strict mode is enabled (`strict: true`, `noUncheckedIndexedAccess`, `noImplicitOverride`).
- Always use explicit types on function parameters and return values.
- Use `interface` for object shapes, `type` for unions and aliases.
- All shared types live in `src/types.ts`. Do not scatter type definitions
  across implementation files.
- Use `as const` for readonly arrays and literal objects.
- Use `type` imports (`import type { ... }`) for types-only imports.
- Never use `any`. Use `unknown` and narrow explicitly.
- Use optional chaining (`?.`) and nullish coalescing (`??`) over defensive
  `if` chains where readable.
- Index access on arrays/records always accounts for `undefined`
  (`noUncheckedIndexedAccess` is on) — use `?? fallback` or guard explicitly.

### Imports
- Node built-ins use the `node:` prefix (`import { randomUUID } from "node:crypto"`).
- Group imports: `node:*` → third-party → internal, separated by blank lines.
- Use relative imports for internal modules (`../constants`, `./response`).
- No barrel `index.ts` files.

### Naming
- `camelCase` for variables and functions.
- `PascalCase` for types, interfaces, and classes.
- `SCREAMING_SNAKE_CASE` for top-level constants in `constants.ts`.
- Route handler files export a named `Hono` instance: `export const chatRoutes = new Hono()`.
- Boolean variables/functions use `is`/`has`/`can` prefix where natural.

### Error Handling
- Functions that can fail return discriminated unions:
  `{ ok: true; ... } | { ok: false; error: string }`.
- Never throw across module boundaries — return `ok: false` instead.
- Use `try/catch` at I/O boundaries (fetch, file read/write); catch blocks
  return the error union or `null`/`undefined` as appropriate.
- HTTP error responses always follow OpenAI error shape:
  `{ error: { message, type, code } }`.
- Use `console.error` for server-side failures, `console.warn` for
  degraded-but-recoverable situations.

### Formatting
- 2-space indentation.
- Double quotes for strings.
- Trailing commas in multi-line objects and arrays.
- Semicolons always present.
- Opening brace on the same line as the statement.
- Max line length is not enforced mechanically but keep lines readable.

### Constants
- All magic strings, URLs, model IDs, and numeric thresholds go in
  `src/constants.ts`. Never inline them in implementation files.

### Zod
- Request body validation uses Zod in route handlers (`safeParse`, not `parse`).
- Schema definitions live at the top of the route file, not inline in handlers.

### Streaming
- SSE responses use `ReadableStream<Uint8Array>` with manual `TextEncoder`/
  `TextDecoder`. Do not use Node.js streams.
- Always emit `data: [DONE]\n\n` as the final SSE event.

---

## Key Invariants

- `src/types.ts` contains types only — no logic, no imports with side effects.
- `src/constants.ts` contains values and pure config helpers — no I/O.
- The token store (`tokens.json`) is read/written exclusively through
  `src/store/tokens.ts`.
- Only the first `system` message in an OpenAI request becomes Gemini's
  `systemInstruction`. Subsequent system messages are silently dropped.
- Gemini requires strictly alternating `user`/`model` turns; consecutive
  same-role turns are merged in `src/gemini/request.ts`.
- Remote image URLs are not supported — only `data:` base64 URIs.

---
> Source: [KashifKhn/gemini-proxy](https://github.com/KashifKhn/gemini-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
