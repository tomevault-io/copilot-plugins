## reader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository identity

This is the `oss` branch of `reader-saas`, the codebase behind `https://r.jina.ai` (URL → markdown) and `https://s.jina.ai` (search → markdown). It is published to https://github.com/jina-ai/reader. The MongoDB-backed SaaS storage layer is **not** in this branch — only the stateless and bucket-cached storage modes ship here.

Companion docs in this repo:
- `architecture.md` — engines, formatting profiles, abuse mitigation, deployment topology.
- `CONTRIBUTING.md` — full env var table, dev workflow, test policy.
- `cookbooks.md` — header recipes for common downstream pipelines (RAG, embedding, deep research, uploads, etc.).

## Common commands

Build / run:
- `npm run build` — runs `integrity-check.cjs` (requires `licensed/GeoLite2-City.mmdb` to exist) then `tsc -p .`. The integrity check is **not** optional — bare `tsc` will skip it.
- `npm run build:watch` — incremental build for the F5 debug flow.
- `npm start` — runs the compiled crawler entrypoint (`build/stand-alone/crawl.js`).
- `npm run dry-run` — `NODE_ENV=dry-run` boots `search.js`, resolves the DI graph, then exits via `finalizer.terminate()`. Used in the Dockerfile to warm `NODE_COMPILE_CACHE`.

Local services (`docker compose up -d`):
- `minio` on `:9000` (API) / `:9001` (console) — also aliased as `minio.dev.jina.ai`. Only needed when running with `BucketStorageLayer`.

VSCode F5 launches `Debug Stand Alone Crawl` (or `Search` / `SERP`), which runs `Backend:prepare` (docker compose + tsc watch) first. `--env-file=.secret.local` is loaded by Node directly — that file is gitignored and must be created locally.

Three independent stand-alone servers:
- `build/stand-alone/crawl.js` — mounts `CrawlerHost` (the `r.jina.ai` surface).
- `build/stand-alone/search.js` — mounts `SearcherHost`. At startup it deletes any registry entries tagged `'crawl'`, so search and crawl are mutually exclusive on a single process.
- `build/stand-alone/serp.js` — SERP-only.

Linting: `npm run lint` (eslint over `.js`/`.ts`).

## Tests

The repo uses the **Node.js built-in test runner**. Do not introduce Jest, Vitest, or similar.

- `npm run test:unit` — pure TS unit tests under `tests/unit/`. Compiled into `tests-build/` before running. No Docker required.
- `npm run test:e2e` — boots the real `CrawlStandAloneServer` via `serviceReady()` and hits `tests/e2e/*.test.ts` against it. Needs Docker services up and `.secret.local` configured.
- `npm test` — runs unit then e2e.
- `npm run test:coverage` — c8 across both suites; merges coverage from unit and e2e runs (`--no-clean` between them).

Single test:
```bash
tsc -p tests/tsconfig.json
node --test tests-build/unit/<file>.test.js
node --test --test-name-pattern '<regex>' tests-build/unit/<file>.test.js
```
For e2e, the test runner expects the crawl server already initialized — easier to run the full file via `node tests-build/run.js` after editing `run.ts` to filter, or just run the whole suite.

The e2e harness shuts down via `finalizer.teardown()` once the enqueued count matches the completed count; if you add async tests that don't go through `node:test`'s lifecycle, the process won't exit cleanly.

## Architecture

### DI: tsyringe + civkit

Every service is a `@singleton()` registered against the global `tsyringe` container. The graph is wired by side-effect: importing a module is what registers it. The conventional bootstrap is:

1. `import 'reflect-metadata';`
2. `import '../config';` — sets `AUTH_DTO_CLS` and `STORAGE_CLS` based on env vars (see below).
3. `container.resolve(...)` to get a fully-injected instance.

`AsyncService` (from `civkit`) is the base. Services emit `'ready'` after `dependencyReady()` resolves. `serviceReady()` waits until the entire graph is initialized. Always wait on `serviceReady()` before listening / running tests.

Don't construct services with `new` — go through `container.resolve` (or constructor injection). Adding a new service means: `@singleton()` + add it to a constructor that's already in the graph (or call `container.resolve` once).

### `src/config.ts` — runtime mode selection

The same code runs in two storage modes. `config.ts` swaps the implementation classes at import time:

- `AUTH_DTO_CLS`: `BaseAuthDTO` (the oss branch always uses the base DTO — no SaaS auth).
- `STORAGE_CLS`: defaults to `StorageLayer` (`db/noop-storage.ts` — every method returns nothing). With `GCP_STORAGE_*` → `BucketStorageLayer` (Stage 1: bucket-only cache).

This mirrors the "Progressive Clustering" stages in `architecture.md`. The noop layer means the same code paths exist in stateless mode — handlers always call `storage.findPageCache` etc., and the noop returns `undefined` so the request just falls through to a live fetch. Don't add `if (storage)` guards; rely on the noop.

When you change a method on the storage layer, you must update **both** implementations: `db/noop-storage.ts` and `db/bucket-storage.ts`.

### RPC routing: civkit + Koa

`src/services/registry.ts` is the routing core. It re-exports `Method`, `RPCMethod`, `Param`, `Ctx`, `RPCReflect` decorators and the `Context` type. Don't import these from `koa` or `civkit/civ-rpc` directly — go through `services/registry.ts` so you get the configured `RPCRegistry` and `ReaderEnvelope`.

`ReaderEnvelope` content-negotiates errors: JSON, text/markdown, and SSE branches all live there. If you add a new content type, add an error path here too — otherwise errors will fall back to the JSON envelope.

Endpoints are `RPCHost` subclasses in `src/api/`:
- `crawler.ts` — `CrawlerHost`, the `r.jina.ai` surface.
- `searcher.ts` — `SearcherHost`, the `s.jina.ai` surface (calls back into crawler).
- `serp.ts` — pure SERP.

### Worker threads

`ThreadedServiceRegistry` (`src/services/threaded.ts`) extends `civkit`'s `AbstractThreadedServiceRegistry`. CPU-heavy work (DOM manipulation, PDF parsing, markify) runs in worker threads. The registry sniffs hyperthreading and sets `maxWorkers` to `cpus.length` or `cpus.length / 2` accordingly. Workers share state via `PseudoTransfer`; do not use `postMessage` directly.

To make a method run in a worker, decorate it with `@Threaded()` — exported by `services/threaded.ts` alongside the RPC decorators. See `services/jsdom.ts` for the canonical pattern (its narrowing/parsing methods are all threaded).

### Engines and formatting profiles

The crawler is multi-engine and multi-profile:
- Engines (URL→HTML): `puppeteer.ts` (browser), `curl.ts` (curl-impersonate), `cf-browser-rendering.ts`. `auto` is the default and is implemented in `crawler.ts`.
- Formatting profiles (HTML→Markdown): `@mozilla/readability`, the `markify` rule-based engine (`services/markify.ts`), and `lm.ts` (ReaderLM v2).
- Header `x-respond-with` selects the profile; `x-engine` selects the engine.

The full surface — including `x-respond-timing`, `x-retain-images`, `x-retain-links`, `x-markdown-chunking`, and the body-field equivalents — lives in `src/dto/crawler-options.ts`. When users ask "what does header X do," that file is authoritative. `cookbooks.md` shows curated combinations.

When adding a new engine or profile, wire it into `CrawlerHost.crawl()` (the dispatch lives there) and register a singleton service for the implementation.

### Auth and rate limiting

Every public method takes an `auth: AUTH_DTO_CLS`. In this branch the base DTO accepts any request — there is no upstream auth check. Rate limiting goes through `storage.rateLimit(ctx, rpcReflect, auth)`, which the noop layer answers with an empty policy. The SaaS-mode enforcement lives outside this branch.

## Environment variables

The full table lives in `CONTRIBUTING.md`. Two shapes you'll see in practice:

- `.secret.local` — gitignored, loaded via Node's `--env-file` flag from `.vscode/launch.json`. Compiled / extracted by `bin/compile-secret.js` and `bin/extract-secret.js`.
- `SECRETS_COMBINED` — base64 JSON blob, the production shape. Decoded and merged by `services/envconfig.ts`.

Mode-selecting vars worth remembering when reading code: `GCP_STORAGE_ENDPOINT` + `GCP_STORAGE_BUCKET` (Stage 1 bucket cache), `NODE_ENV=dry-run` (DI-graph warmup, used in the Dockerfile), `DEBUG_BROWSER=true` (non-headless puppeteer), `JINA_CRAWLER_OFFLOAD_ORIGIN` (search → peer crawler cluster).

## Conventions specific to this repo

- Decorators (`@singleton`, `@Method`, `@Param`, `@Threaded`, etc.) require `experimentalDecorators` and `emitDecoratorMetadata` — both already on. Do not migrate to TC39 decorators.
- The `licensed/` folder holds external (non-redistributable) artifacts (GeoLite mmdbs, Source Han Sans font, gsa user-agent list). It's gitignored. CI fetches them inline; locally, run `npm run assets:download` (or `bash ./download-external-assets.sh` directly) — idempotent, skips files already present. The build's integrity check requires `licensed/GeoLite2-City.mmdb` to exist; if `npm run build` errors on it, you forgot to run the download. The folder is still named `licensed/` for source-code/Dockerfile compatibility — only the script and env vars use the "external" naming.
- Build output is `build/`, test build output is `tests-build/`. Both are gitignored. The Dockerfile copies pre-built `build/` rather than building inside the image.
- HTTP/2 cleartext (h2c) is used in production; `crawl.ts` / `search.ts` install an `http2.createServer` on top of the Koa callback. Each request gets its own `traceId` (random UUID) because h2c connections multiplex — don't rely on `x-cloud-trace-context` for per-request identity.
- Public assets in `public/` are served directly by the stand-alone server's `walkForAssets()` — drop a file in there, rebuild, it's served at `/<filename>`.
- The Reader API surface is entirely header- and body-driven; there are no path-based routes for options. When changing behavior, the right place is almost always `src/dto/crawler-options.ts` (parsing) plus `src/api/crawler.ts` (dispatch).

---
> Source: [jina-ai/reader](https://github.com/jina-ai/reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
