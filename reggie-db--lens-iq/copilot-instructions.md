## dry-this-repo

> DRY + library-first conventions for this repo. Read before writing or refactoring code.


# DRY playbook for lens-iq

Always grep / read before writing. Most "new code" in this repo has a
helper that already exists. If you find yourself writing one of the
patterns below, stop and reuse / extract.

## Server helpers (in `server/util.ts` unless noted)

| Need | Helper |
| --- | --- |
| Async one-shot init (table/index bootstrap) | `onceAsync(fn)` |
| Best-effort DDL that tolerates ownership skips | `_runIdempotentDdl(sql)` in `server/server.ts` |
| Validate + handle a route, throw on bad input, JSON-encode the return | `asyncRoute({body, query, params}, handler)` |
| Multi-row INSERT with `$N` placeholders | `buildBatchInsert(table, columns, rows)` |
| Cap a base64/data-URL payload before persisting to Postgres | `inlineBlob(value)` |
| Return a typed HTTP error from a route | `throw new HttpError(status, message)` |
| Stream a file from disk or volume | `_streamLocal*` / `_streamVolume*` in `server/server.ts` |
| JSON fetch with typed errors (client) | `fetchJson()` in `client/src/lib/serving-status.ts` |
| Detection / sample-video plumbing (client) | `useDetectionLoop` / `useSampleVideoStream` / `useWebcamStream` |

The error middleware is mounted at the end of `onPluginsReady`. It
turns the following into the right HTTP response:

- `HttpError` -> its status + `{error: message}`
- `ZodError` -> 400 with the offending field path
- `EndpointNotDeployedError` -> 503 envelope the UI branches on
- anything else -> 500 with the thrown message

That means **routes should `throw`** instead of writing
`res.status(500).json(...); return;`. The whole point of
`asyncRoute` is that the per-route try/catch goes away.

## Library-first (don't re-implement what's installed)

1. **AppKit plugins.** Before talking to Databricks, check:
   ```bash
   npx @databricks/appkit plugin list
   npx @databricks/appkit docs --full
   ```
   GA + beta export lists live at
   `node_modules/@databricks/appkit/dist/plugins/{ga,beta}-exports.generated.d.ts`.
2. **AppKit pool is the only Postgres pool.** Every SQL statement goes
   through `appkit.lakebase.query(sql, params)`. Do not import
   `pg`, `postgres`, `drizzle-orm/node-postgres`, `kysely`, or any
   client that spins its own `pg.Pool` from `getOrmConfig()` /
   `getPgConfig()`. A second pool defeats AppKit's OAuth refresh and
   graceful drain and burns connection slots on the instance.
3. **UI primitives** come from `@databricks/appkit-ui/react`: `Empty`,
   `Skeleton`, `Spinner`, `Alert`, `Toaster`, `Card`, `Badge`,
   `Button`, etc. The toast notification system is `Toaster` (mounted
   in `client/src/main.tsx`) backed by `sonner` (`import { toast }
   from "sonner"`). Do not add `react-hot-toast` back.
4. **Validation** is `zod`. Every new route gets a schema, not a
   `typeof body?.x === "string"` ladder.

## Always-apply checklist (before sending the diff)

1. **Did you `throw new HttpError(...)` instead of
   `res.status(...).json({error}).return`?** If you wrote a
   per-route try/catch envelope, delete it - `asyncRoute` already
   forwards to the error middleware.
2. **Did you use a zod schema?** No new `typeof body?.x === "string"`,
   `Number(req.query.limit ?? 50); Number.isFinite ? Math.min(...) : 50`,
   or `if (!body || !body.x) { res.status(400)... }` chains. Schema
   defaults handle the limit clamp:
   `z.coerce.number().int().min(1).max(200).default(50)`.
3. **Did you use `buildBatchInsert` for the multi-row INSERT?** Don't
   hand-roll `($1,$2),($3,$4)` placeholders.
4. **Did you `onceAsync(...)` the bootstrap?** No new boolean
   `*Ensured` flags. No bare `await _ensureXTable()` chains that
   re-run the full DDL per request.
5. **Did you wire `_ensureXTable()` through `_runIdempotentDdl`?**
   Don't `await appkit.lakebase.query(CREATE_INDEX_...)` directly -
   it can't swallow the ownership skip and will 500 in local dev.
6. **Did you reuse `inlineBlob()` for the 750KB cap?** Three callers
   today, growing.
7. **One source of truth for catalogs.** `MODELS`, `SAMPLE_VIDEOS`,
   `PRESENTER_CONTENT` are imported by both server and client. New
   runtime catalogs go in `client/src/lib/*.ts` and are imported
   from `server/server.ts`.
8. **Magic numbers that repeat are constants.** Cap repeated values
   (`PG_INLINE_BLOB_MAX`, `FACE_MATCH_DEDUP_MS`,
   `FACE_MATCH_DEFAULT_THRESHOLD`, `SSE_HEARTBEAT_MS`,
   `POLL_INTERVAL_MS`) - don't inline `750_000`, `30_000`, `0.45`
   anywhere new.
9. **Scripts share `scripts/_sync-lib.sh`.** Any new
   `scripts/sync-*.sh` must source it and use `sync_parse_target` /
   `sync_resolve_volume` / `sync_run`.
10. **SSE routes share the same skeleton.** `/api/detections/stream`
    and `/api/face-matches/stream` are 95% the same. If you add a
    third, extract `streamSse(req, res, { onTick, intervalMs,
    heartbeatMs })` into `server/util.ts` first.

## Anti-patterns to refuse on sight

- `let _xxxTableEnsured = false;` flag - use `onceAsync`.
- `catch (err) { res.status(500).json({error: ...}); return; }` - throw
  `HttpError` or just `throw err` and let the middleware respond.
- `typeof body?.x === "string" ? ... : null` ladder - zod schema.
- `Number(req.query.X ?? D); Number.isFinite ? Math.min(...) : D` - 
  zod schema with `z.coerce.number().int().min().max().default()`.
- `console.warn("...failed:", err)` followed by silently returning -
  either re-throw or document the swallow (the only sanctioned
  swallow today is `_runIdempotentDdl`'s ownership skip).
- New `pg` / `pg.Pool` import, new `postgres` (postgres.js), new
  `drizzle-orm` setup, new `kysely`. All Postgres goes through
  `appkit.lakebase.query()`.
- New `react-hot-toast` import - use `sonner`'s `toast`; the `Toaster`
  is already mounted from `@databricks/appkit-ui/react`.
- New `useEffect` + `setInterval` + manual cleanup in a page - reach
  for the existing `useDetectionLoop` / `useSampleVideoStream` style
  hook first, or propose adding `@tanstack/react-query`.
- Hand-rolled `($1, $2, $3), ($4, $5, $6)` placeholder builder - use
  `buildBatchInsert`.

## When you do extract a helper

- Lives next to the second caller's file unless it has 3+ callers,
  then promote to `server/util.ts` or `client/src/lib/util.ts`.
- Stays small. Five-line helpers are fine. If it's growing parameters
  to fit two callers, it's the wrong abstraction - inline and rethink.
- Has a comment explaining *why* it exists (the race, the ownership
  skip, the dedup window, the AppKit pool constraint). The how is in
  the code.

---
> Source: [reggie-db/lens-iq](https://github.com/reggie-db/lens-iq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
