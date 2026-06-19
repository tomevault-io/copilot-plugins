## console-tanstack-api

> >-


# Console TanStack Query + server functions

Use this pattern whenever the console hits the network or server state from UI code.

## Server entrypoints

Put `createServerFn` (TanStack React Start) handlers in **`*.functions.ts`** next to related auth or API helpers. Prefer one module per bounded context (example: `session.functions.ts`, `api-auth.functions.ts`, `integrations/bsky/bsky-typeahead.functions.ts`).

## Client consumption

React components (and hooks they use under `packages/console/src/components/`) **must not** invoke server functions returned from `createServerFn` directly. They must use **`queryOptions`** and **`mutationOptions`** from `@tanstack/react-query` only:

| Need | Export from `*.functions.ts` |
|-----|-------------------------------|
| Read / cache | `export const xyzQueryOptions = queryOptions({ ... })` |
| Writes / imperative calls | `export const xyzMutationOptions = mutationOptions({ ... })` |
| Stable query key factories | Same module: `xyzQueryOptions` or helpers like `bskyHandleTypeaheadQueryOptions(q)` |

Use `useQuery(frozenOptions)`, `useMutation({ ...options, onSuccess })`, or **`queryClient.fetchQuery(options)`** for one-shot reads in event handlers — still only through exported options objects.

Deep links (`beforeLoad`) and loaders may call a **named** `createServerFn` export from the same file (exported as `*ServerFn`). Keep that naming so it is clearly not for client JSX.

## Effect placement

Outside of `.handler(...)`, **`Effect`** programs and adapters live in **`*.server.ts`** (or route `server.handlers` / **`middleware/*.server.ts`**). Do not import `effect` into files that compile for the browser unless the import is erased by the bundler via a documented pattern (default: assume it leaks). Client components route work through **`*.functions.ts`** server functions instead.

## Client bundles must not import `*.server.ts`

Files marked **`"use client"`** (and any module that ships to the browser) **must not import** from **`*.server.ts`**, even for types or “pure” helpers. Those modules are for the server boundary, AppView/auth adapters, and Effect.

- Put **serializable types**, **pure transforms**, and **display math** shared by handlers and UI in **`*.shared.ts`** next to the feature (or under `lib/` without the `.server` suffix).
- **Aggregates over fetched data** that the UI needs for a dashboard (lifetime totals, counts across the full `rows` array, etc.) belong in the **`createServerFn` handler** in **`*.functions.ts`**, returned on the **query/mutation payload**. The client reads them from **`useQuery` data**—do not recompute the same aggregate in the client by importing server modules.
- The client may still **derive purely local views** from query data (e.g. filter by a **client-only** UI control) using **`*.shared.ts`** helpers and **`useMemo`**—without calling `createServerFn` or importing `*.server.ts`.

---
> Source: [graze-social/cocore](https://github.com/graze-social/cocore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
