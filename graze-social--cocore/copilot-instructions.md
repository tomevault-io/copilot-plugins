## console-effect-ts

> Use Effect (`effect`) for server-side API workflows in `@cocore/console`


# Console server API and Effect-TS

All **server** work in the paths above (TanStack Start `createServerFn` handlers, route `server.handlers`, device-pair XRPC helpers, auth middleware under `middleware/*.server.ts`) should be implemented as **Effect programs** and executed at the framework boundary with `Effect.runPromise` or `Effect.runSync`.

## Patterns

- Model orchestration with `Effect.gen` and `yield*`; for third-party **Promise** APIs, add a dedicated `*Effect` next to the adapter in a **`*.server.ts`** module (e.g. `atprotoOAuthAuthorizeEffect`, `requestJsonEffect`) using `Effect.async`, then `yield*` that from programs. Use `Effect.try` for synchronous code that throws.
- Keep **small route files**: put reusable programs next to adapters in **`*.server.ts`** (for example under `integrations/auth/`) when it keeps routes thin.
- At the HTTP/TanStack boundary, return `Promise<...>` or `Response` by running the effect — do not leave long `async`/`await` chains in handlers when the logic is non-trivial.
- **Do not** add raw `try`/`catch` in route handlers for control flow if the same can be expressed as `Effect.either` / `Effect.catchAll` inside a program (OAuth callback is the reference for failure-to-redirect mapping).

## Imports

```typescript
import { Effect, Either } from "effect";
```

Avoid `Effect.promise` and **`Effect.tryPromise`** (and ad-hoc generic Promise bridges) in `@cocore/console`. Prefer **named** `*Effect` functions colocated with the async source, `Effect.either` where failures should map to values, and `Effect.catchAll` when swallowing errors is intentional.

---
> Source: [graze-social/cocore](https://github.com/graze-social/cocore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
