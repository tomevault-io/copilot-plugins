## dynamic-worker-based-reports

> Guidance for AI coding agents working on this project.

# AGENTS.md — Dynamic Workers Report Builder

Guidance for AI coding agents working on this project.

---

## What This Project Does

A demo app for the "Run Code That Writes Itself" YouTube video. A user types a natural-language prompt, Workers AI generates a JavaScript Cloudflare Worker, that Worker is compiled and run as a Dynamic Worker with a controlled data binding, and an HTML report with an interactive chart appears.

The key Cloudflare primitives in play:

| Primitive | Binding | Purpose |
|---|---|---|
| Workers AI | `env.AI` | Generates the report Worker code from a prompt |
| Worker Loader (Dynamic Workers) | `env.LOADER` | Compiles and runs AI-generated code at runtime |
| Workers RPC / WorkerEntrypoint | `DataService` via `ctx.exports` | Exposes the survey dataset to the dynamic worker |
| KV | `env.REPORTS` | Stores saved report metadata + source code |
| Durable Objects | `env.LOG_SESSION` | Log capture pipeline (implemented, not yet wired to LOADER) |

---

## Architecture

```
Browser (React + Vite)
    │
    └── POST /api/run  (SSE stream)
            │
            ▼
    Loader Worker  (src/index.ts)
            │
            ├── env.AI.run()               → nemotron generates JS worker code
            ├── createWorker({ files })    → @cloudflare/worker-bundler bundles it
            ├── env.LOADER.get(id, cb)     → Dynamic Worker loaded and cached
            └── ctx.exports.DataService() → RPC binding injected into dynamic worker
                        │
                        ▼
            Dynamic Worker  (AI-generated)
                ├── env.DATA.getAIData()  → fetches survey data via RPC
                └── returns HTML page with Chart.js CDN charts (client-side rendering)
```

Saved reports are stored in KV under two keys:
- `id:<workerId>` — looked up by list/kill endpoints
- `slug:<slug>` — looked up by `/r/:slug` route

On a cache miss (isolate evicted), the loader re-bundles from the `fullCode` stored in KV.

---

## File Structure

```
src/
  index.ts              # Loader Worker — all API routes + Dynamic Worker orchestration
  data-service.ts       # DataService WorkerEntrypoint + 82-record SO survey dataset
  log-session.ts        # LogSession DO + DynamicWorkerTail (log capture pipeline)
  main.tsx              # React entry point
  ui/
    App.tsx             # Root component — state, API calls, layout
    PromptPanel.tsx     # Left panel: prompt input, binding toggle, Run/Save
    OutputPanel.tsx     # Right panel: iframe report, generated code tab, logs, timing
    ReportList.tsx      # Bottom panel: saved report cards with slug URLs
wrangler.jsonc          # Worker config
vite.config.ts          # Vite config (proxies /api/* and /r/* to Worker in dev)
env.d.ts                # Type references for wrangler-generated types
```

---

## Running Locally

Two terminals are required — Dynamic Workers and Workers AI don't work with the local Wrangler simulator, so the worker runs against real Cloudflare infrastructure.

```bash
# Terminal 1 — Vite dev server (UI, proxies API calls to Worker)
npm run dev:ui

# Terminal 2 — Wrangler dev against Cloudflare (remote mode)
npm run dev:worker
```

Open `http://localhost:5173`.

---

## Deploying

```bash
npm run deploy          # vite build + wrangler deploy
```

The `REPORTS` KV namespace is auto-provisioned on first deploy if it doesn't exist (no `id` in `wrangler.jsonc`). After first deploy, wrangler writes the real ID back into the config.

After any change to `wrangler.jsonc` bindings, regenerate types:

```bash
npm run types           # wrangler types
```

---

## Key Implementation Details

### Dynamic Worker Lifecycle

1. **Generate** — `env.AI.run(nemotron, { messages, stream: true })` streams the worker code. If streaming returns empty (nemotron quirk), a non-streaming fallback call is made.
2. **Strip** — `buildDynamicWorker()` removes any `import` statements the model may have added. The generated code must be self-contained.
3. **Bundle** — `createWorker({ files: { "src/index.js": code } })` from `@cloudflare/worker-bundler` compiles and bundles the code.
4. **Cache** — `env.LOADER.get(workerId, callback)` returns a warm isolate if the same worker ID was seen recently. Worker IDs are content-hashes of the generated code + binding flag, so the same prompt + same binding always hits the warm cache.
5. **Run** — `worker.getEntrypoint().fetch(request)` executes the dynamic worker.

### Why Chart.js (not vega-lite)

Vega/vega-lite were attempted but are incompatible with the Dynamic Worker runtime. The `@cloudflare/worker-bundler` output is a CJS/ESM hybrid; vega's transitive dependencies import Node built-ins (`stream`, `node:http`) as bare ESM specifiers. With `nodejs_compat` enabled, the esbuild `__require` shim bypasses its bundled registry and calls the real `require()`, which can't find `vega-lite`. Without it, the Node built-in imports fail. No combination of compatibility flags resolves both.

Chart.js avoids this entirely: the dynamic worker returns plain HTML with a `<script src="https://cdn.jsdelivr.net/npm/chart.js@4/...">` tag, and all charting happens client-side in the browser. The iframe has `sandbox="allow-scripts"` to permit this.

### The DATA Binding

`DataService` is a `WorkerEntrypoint` (in `src/data-service.ts`) that exposes `getAIData(filters?)`. It's instantiated per-request via `ctx.exports.DataService({ props: {} })` and passed into the dynamic worker's `env.DATA`. The dynamic worker never gets direct access to the host worker's bindings — only what's explicitly handed to it through `env`.

When `withBinding` is false (the intentional failure demo), `env.DATA` is not passed, and the dynamic worker throws when it tries to call `env.DATA.getAIData()`. This is capability-based sandboxing demonstrating that the dynamic worker only has access to what it's explicitly given.

### Slug-Based Report URLs

On save, a human-readable slug is generated from the prompt (`makeSlug()` in `src/index.ts`) with a 6-char worker ID suffix to avoid collisions. Reports are accessible at `/r/<slug>`. The full assembled worker source (`fullCode`) is stored in KV so reports can be re-bundled after an isolate eviction.

### Log Capture Pipeline

`LogSession` (Durable Object) and `DynamicWorkerTail` (`WorkerEntrypoint`) are implemented in `src/log-session.ts` and fully wired:

- `tails: [ctxExports.DynamicWorkerTail({ props: { workerId } })]` is passed in the `LOADER.get()` callback
- `moduleExports.LogSession.getByName(workerId).waitForLogs()` returns a `LogWaiter` `RpcTarget` before the dynamic worker runs
- After `entrypoint.fetch()` returns, the loader waits up to 5s via `logWaiter.getLogs(5000)` for the tail to deliver
- If logs arrive they are sent as a trailing `logs` SSE event; the UI handles this event and updates the log panel

In practice the AI-generated workers have no `console.log` calls so the log panel will typically be empty on successful runs. Exceptions thrown by the dynamic worker DO surface as `exception`-level log entries. The pipeline is verified against the [Dynamic Workers observability docs](https://developers.cloudflare.com/dynamic-workers/usage/observability/).

---

## Data

Stack Overflow Developer Survey 2024 and 2025, licensed under ODbL. 82 records across 41 countries × 2 years. Only countries with n ≥ 100 respondents are included.

Source: https://survey.stackoverflow.co/
License: https://opendatacommons.org/licenses/odbl/

Fields: `country`, `region`, `respondents`, `aiUsagePct`, `favorablePct`, `skepticalPct`, `trustPct`, `year`

Notable patterns in the data (useful for demo prompts):
- Ukraine leads Europe on usage both years (72% → 85%)
- Iran leads all countries in 2025 (87.9%)
- Nigeria leads on trust in 2025 (68.2%)
- US/UK/Canada/Australia skepticism roughly tripled 2024→2025
- Colombia leads Americas on usage in 2025 (86.6%)

---

## What Not to Break

- `wrangler.jsonc` `migrations` — the `LogSession` DO uses `new_sqlite_classes`. Changing the migration tag or class name requires a new migration entry.
- Worker IDs must remain stable for warm caching. `makeWorkerId()` hashes `fullCode + bindingFlag`. Any change to how `fullCode` is assembled busts all existing caches (fine) but also invalidates existing saved report isolates (they re-bundle from KV on next load, which also fine).
- The `sandbox="allow-scripts"` attribute on the iframe in `OutputPanel.tsx` is required for Chart.js to run. Do not remove it or add `allow-same-origin` (which would allow the report to access the parent page's DOM/cookies).
- KV keys use `id:` and `slug:` prefixes to avoid collisions between the two lookup paths. The `handleList` function lists only `id:` prefix keys. Don't change the prefix scheme without migrating existing data.

---
> Source: [craigsdennis/dynamic-worker-based-reports](https://github.com/craigsdennis/dynamic-worker-based-reports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
