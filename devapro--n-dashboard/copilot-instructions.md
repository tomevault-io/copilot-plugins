## n-dashboard

> Notes for anyone — human or agent — working in this repository. `README.md` is the

# CLAUDE.md

Notes for anyone — human or agent — working in this repository. `README.md` is the
user-facing documentation; this file is the parts that are easy to break.

## What this is

A personal dashboard for a trusted LAN. One idea carries the whole design: **a
widget is a TypeScript function on disk**. The platform decides *when* to call it
(clock, MQTT message, webhook), validates *what* it returns against a typed
schema, and pushes the result to every open tab over SSE. There is no plugin
format and no config-driven renderer path, so summing a field, joining two APIs
and remembering what you already saw are all the same amount of work.

Widget code is **trusted**. Isolation exists to contain bugs, not attackers. There
is one optional login, over `/admin` only (`ADMIN_PASSWORD` in `.env`) — it keeps
the panel out of reach of whoever walks past the wall display, and it is not a
hardening story. Multi-user is still out of scope; see the bottom.

## Commands

```bash
npm run dev        # node --watch on :8080 + vite on :5273 (proxies /api)
npm test           # vitest; MQTT suites skip themselves without a broker
npm run typecheck  # both projects; run this, tsc catches what vitest can't
npm run build      # web bundle only — the server has no build step

docker compose -p n-dashboard-dev -f docker/compose.dev.yml up -d   # broker for tests
```

## Repo map

```
.claude/skills/ write-widget — the widget-authoring workflow, for agents
src/shared/    ctx.d.ts (the widget API), kind schemas, duration parsing
src/server/    express, worker pool, scheduler, connections, ndjson store
src/web/       react — dashboard (kept light) + admin (Monaco, lazy-loaded)
docker/        Dockerfile, both compose files, broker configs
docs/          the long-form guides linked from README.md, plus the Pages page
examples/      seeded into data/ disabled on first run
data/          runtime state, gitignored
```

## Invariants

Break one of these and it usually fails somewhere far away from the change.

**`src/shared/ctx.d.ts` is the single source of truth for the widget API.** It is
served verbatim at `/api/widget-api.d.ts` for Monaco, imported as types by both
the server and the web app, and quoted in the README. It must stay a plain
ambient `declare module` — no top-level import or export — or the ambient
declaration stops applying.

**Kind schemas are declared `z.ZodType<KindDataMap[K]>`.** That is what makes a
disagreement between `ctx.d.ts` and a runtime validator a compile error rather
than a widget that validates against a shape the editor never offered. Keep it.

**`KIND_CATALOG` and the web `RENDERERS` map are declared over the full
`KindName` union.** Adding a type to `ctx.d.ts` without registering a schema and
a renderer is therefore a compile error. This is on purpose.

**Results are validated on the main thread, after the worker returns.** Never
inside the worker: a widget must not be able to vote on whether its own output is
valid.

**Runtime aliases are package.json `imports` (`#shared/*`, `#server/*`), not
tsconfig `paths`.** tsconfig paths are a type-only fiction that Node cannot
resolve; subpath imports resolve natively in every thread, which is what lets
worker threads run with no loader registered. Consequences:

- Server and shared code must use **explicit `.ts` extensions** in relative and
  `#alias` imports.
- `node --watch-path=./src` is deliberate. Without the path restriction, `--watch`
  also watches the dynamically imported `data/widgets/*/widget.ts` and restarts
  the server every time the admin panel saves a widget.

**Everything obeys `erasableSyntaxOnly`** — no `enum`, no `namespace`, no
constructor parameter properties, anywhere. Widget code is run by Node's type
stripping, and the rest of the project matches so the rules are uniform.

**Widget modules are cache-busted with `?v=<mtimeMs>`.** A worker that has
imported `widget.ts` caches it for the life of the thread, so without the query an
edited widget would keep running its old code until restart.

**Secrets and `ctx.state` ride along in the invoke message.** They are small and
known up front, and shipping them is what lets `ctx.secret()` and
`ctx.state.get()` stay *synchronous* for widget authors despite the thread
boundary. Only `mqtt`, `google` and `history` calls round-trip to the main thread.

**The admin guard is fail-closed: `PUBLIC` in `src/server/auth.ts` is the entire
list of ungated API routes.** A route added anywhere under `/api` is protected the
moment `ADMIN_PASSWORD` is set, until somebody names it there — the alternative,
listing what to protect, ships an open route every time a file is added. The list
is read-only: a signed-out board draws itself and a webhook fires, and every
write — layout, editing, theme, refresh — needs a session. That is what the
dashboard header's `signedIn` checks mirror; they hide buttons that would 401, they
do not decide anything. `authRouter` is mounted *before* the guard in `index.ts`,
which is why the login endpoints work while signed out; they are not in `PUBLIC`.
The React `AdminGate` is likewise a convenience so the panel shows a form instead
of a screen of failed requests — it is not the boundary, and nothing may start
depending on it as one.

**Never add `compression` middleware.** It buffers SSE. The events route also
sets `no-transform` and `X-Accel-Buffering: no` so dev and reverse proxies don't
buffer it either.

**`event: error` collides with `EventSource`'s own error event.** Both dispatch as
type `"error"` in the browser. They are told apart by payload: a frame is a
`MessageEvent` carrying `data`, a dropped connection is a bare `Event`. See
`src/web/lib/sse.ts`.

**There is one board, and every open tab shows the same state of it.** The layout,
`editing` *and* `theme` all live in `dashboard.json`, and every route that writes
them calls `emitConfig()`; tabs refetch on the `config` frame rather than diffing.
So you rearrange on a laptop and watch the wall display follow, no tab is left
silently draggable, and you don't walk over to the wall to switch it to dark.
Local component state for any of the three would desynchronise the boards — the
optimistic paint (`setQueryData` for `editing`, `syncTheme` for the theme) is the
only local copy, and the refetch overwrites it. `localStorage` still holds the
theme, but purely as a pre-paint hint so the first frame isn't the wrong palette;
the server's value wins the moment it lands.

**One CSS variable set drives Tailwind, the cards and the ECharts theme.** The
chart theme reads the same custom properties the cards use, so the two cannot
drift. The order of `--chart-1..8` is the colour-blind safety mechanism, not
decoration — it was validated for separation against the card surface in both
light and dark. Do not reorder or substitute without re-validating.

**Backoff is derived from the run history, not stored.** `nextAttemptAt()` reads
`failureCount` and the last run's timestamp, so a widget that succeeds by *any*
route clears its own backoff immediately. Backoff gates the clock only: an MQTT
message, a webhook or a manual refresh always runs.

**The colon separates the two secret namespaces.** `.secrets.json` holds both
connection slots (`type:name:slot`, written only by that connection's form) and
plain API tokens (bare keys, what `ctx.secret('github')` reads, written by the API
tokens panel). `tokenNameSchema` refuses a colon, which is the only thing stopping
the token routes from editing or deleting a connection's credentials. Both routes
are write-only: nothing ever returns a value. Duplicating a connection re-keys its
slots onto the new prefix inside `secrets.ts` — that is the one place a stored
value is read, and it still never crosses the response boundary.

**Config writes go through `writeJsonAtomic`** — temp file, fsync, rename, with
the previous contents kept as `.bak`. History is the exception and appends
directly, because an append must not rewrite the file; it is trimmed on an
amortised schedule instead.

## Library gotchas

These cost real time to find. They are all load-bearing.

**monaco-editor 0.56** exposes the language services at `monaco.typescript`, not
`monaco.languages.typescript`. The old path throws, and because the call sits in
an async setup function the failure is silent — you get an editor with no compiler
options and no injected types.

**The Monaco extra lib must live at a neutral URI** (`file:///n-dashboard-widget-api.d.ts`),
*not* at the module's resolution target under `node_modules/`. `ctx.d.ts` is one
big ambient `declare module`; placed at the resolution target, TypeScript treats
the file itself as the module, which exports nothing, and every widget import goes
red.

**`@monaco-editor/react` fetches Monaco from a CDN** unless pointed at the bundled
copy with `loader.config({ monaco })`. This box is meant to work on a LAN with no
internet. Import the package **root** (`monaco-editor`), not
`monaco-editor/editor/editor.api` — the root is what attaches `typescript` to the
namespace, and importing the pieces separately hands Vite's dep optimizer two
module instances.

**`echarts-for-react`: import from `esm/core`, not `lib/core`.** The `lib` build is
CommonJS and its default export arrives as a namespace object, which React rejects
as an element type. Also: the wrapper **bakes the mount-time `clientWidth` into the
init opts**, so a bare `resize()` re-applies that stale width forever. Since
react-grid-layout animates card widths, every chart locks ~10% too wide unless
resized with `{ width: 'auto', height: 'auto' }`.

**react-grid-layout 2.x is not the v1 API.** `width` is a required prop (no
`WidthProvider`), and options are grouped into `gridConfig` / `dragConfig` /
`resizeConfig`. Use the `useContainerWidth` hook.

**shadcn's `init` rewrites the CSS token set.** It renamed `--muted` from dim text
to a surface. The palette in `src/web/styles.css` is now reconciled: the authored
tokens are the design's own names, and one `@theme inline` block translates them
into the names shadcn's copied-in components expect. Re-running `init` will undo
that.

## Testing notes

- Several stores are **process-global** by design — the runs ring buffer, the SSE
  replay maps, the secrets cache. Tests must call `resetRuns()`, `resetSse()` and
  `resetSecretsCache()` in `beforeEach`, or state leaks between cases and produces
  confusing passes and failures.
- Point `process.env.DATA_DIR` at a fresh `mkdtemp` per test. `paths.ts` resolves
  it on every call precisely so this works.
- The history trim counter is keyed by widget id and lives for the module's
  lifetime; a test that counts appends needs an id no earlier test has touched.
- MQTT suites probe for a broker and skip themselves if it is absent, so `npm test`
  passes without Docker — but then it is not testing MQTT. Start the dev broker.
- No UI tests. For a personal dashboard the eyes are the test; the suites cover
  what is easy to get wrong and hard to notice.

## Conventions

- Comments explain *why*, not what. Several in the codebase record a decision that
  looks arbitrary until you know the alternative that failed.
- The dashboard route is kept light — a wall display loads it. Monaco and the
  admin panel are lazy.
- Errors name the fix: `secret 'slack' is not set — add it under /admin/connections`
  beats `undefined secret`.
- `data/` is the only mutable state. Anything that needs backing up lives there.

## Out of scope

Deliberately absent, not pending. Please don't add them without a conversation:
user accounts, roles and anything multi-user (the `/admin` login is one shared
credential from the environment, not a user model), multiple dashboards or tabs,
alert *delivery*
(alert widgets display state only), IMAP, widget import/export or a sharing
registry, downsampling history to a second resolution tier, and phone-optimised
breakpoints.

---
> Source: [devapro/n-dashboard](https://github.com/devapro/n-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
