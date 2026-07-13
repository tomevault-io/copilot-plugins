## miia

> Lightweight, decorator-driven HTTP framework for TypeScript. Inspired by Angular/NestJS architectural ideas - decorator-driven DI, controllers, modules - Koa's onion-model middleware, and Hono's multi-runtime, lightweight philosophy. Built from scratch on modern standards.

# MiiaJS

Lightweight, decorator-driven HTTP framework for TypeScript. Inspired by Angular/NestJS architectural ideas - decorator-driven DI, controllers, modules - Koa's onion-model middleware, and Hono's multi-runtime, lightweight philosophy. Built from scratch on modern standards.

## Philosophy

- **Web Standards first** - Request/Response API, no Express/Fastify lock-in
- **TC39 native decorators** - no reflect-metadata, no experimental flags
- **ESM-only** - ES2025 target, `nodenext` module resolution, `verbatimModuleSyntax`
- **Runtime-agnostic** - Bun, Deno natively; Node.js/uWebSockets via server packages
- **Minimal abstractions** - Koa-style middleware replaces interceptors/pipes/filters. One concept, full power
- **Flat DI** - per-app container, no module scoping by default. Simple beats "correct"
- **Lightweight schemas** - use Drizzle/Papr/Mongoose schemas directly, no decorator-based ORM layer

## Monorepo structure

```
packages/
  core/          - DI, decorators, router, middleware, exceptions, response, cors, logger
  config/        - ConfigModule, ConfigService, validated env via Zod
  serve-static/  - Static file serving (Range, ETag, charset, SPA fallback, dotfile guard)
  node-server/   - Node.js HTTP server (optimized + native modes)
  uws-server/    - uWebSockets.js HTTP server (optimized + native modes)
  auth/          - Strategy-based auth, JWT (jose), Local
  rate-limit/    - Fixed-window rate limiting: rateLimit middleware, RateLimitGuard, @RateLimit/@SkipRateLimit, pluggable stores
  drizzle/       - Drizzle ORM integration (postgres/mysql/sqlite)
  papr/          - MongoDB integration via Papr
  mongoose/      - MongoDB integration via Mongoose
  swagger/       - OpenAPI 3.1 spec generation, Swagger UI serving
  messaging/         - Decorator-driven message bus (event-bus pattern), retry, DLQ, idempotency, named buses, W3C tracing
  messaging-redis/   - Redis Streams transport for messaging (consumer groups, ZSET retry)
  testing/       - TestApp harness for integration tests
  cli/           - Dev CLI: dev, build, start, check, new commands
examples/
  drizzle-app/   - CRUD with Drizzle + PostgreSQL
  papr-app/      - CRUD with Papr + MongoDB
  mongoose-app/  - CRUD with Mongoose + MongoDB
  full-app/      - Full stack example using auth, drizzle, jwt, swagger, serve-static
  messaging-app/ - Event-driven orders flow with @miiajs/messaging + Redis Streams transport
  uws-app/       - Minimal CRUD on @miiajs/uws-server (Node-only)
apps/
  website/       - Documentation site (Nuxt 4)
```

Benchmarks live in a separate repo: [github.com/miiajs/benchmarks](https://github.com/miiajs/benchmarks).

## Tooling

- **Package manager:** Bun (`bun install`, `bun add`)
- **Test runner:** Bun (`bun test`), tests import from `bun:test`
- **Build:** `tsc --build` with composite project references (`tsconfig.build.json`)
- **Formatter:** Biome - single quotes, trailing commas, semicolons as needed (`biome.json`)
- **Git hooks:** Lefthook - pre-commit auto-formats staged files (`lefthook.yml`)
- **Root scripts:** `build`, `build:watch`, `clean`, `test`, `typecheck`, `format`, `format:check`

## Key patterns

### DI: per-app container with inject()

Each `Miia` instance owns its own `Container`. Use `inject(Token)` in a field initializer to resolve dependencies from the active container during class instantiation.

Three scopes: `singleton` (default), `transient` (new instance per resolve), `request` (per HTTP request, cleared after response).

Lifecycle hooks via duck-typing (no interface required): `onInit(): Promise<void>` called during `container.initAll()`, `onDestroy(): Promise<void>` called during `container.destroyAll()`.

`runInContainerContext(container, fn)` executes a function with a specific container as the active context - used internally by module loader and available for advanced use cases.

### Decorators: Symbol.metadata

`@Injectable`, `@Controller`, `@Module` store metadata via TC39 `Symbol.metadata` (polyfilled in `@miiajs/core`). No WeakMaps, no pending drain.

Metadata helpers: `getMeta()`, `setMeta()`, `pushMeta()`, `addToMapMeta()`, `setInMapMeta()`.

External packages use decorator creators for custom decorators:
- `createClassDecorator()` - class-level (e.g. `@ApiTag`)
- `createMethodDecorator()` - method-level (e.g. `@ApiOperation`)
- `createFieldDecorator()` - field-level
- `createDecorator()` - dual class/method (e.g. `@SkipGuard`, `@ApiSecurity`)

### Module system: @Module and dynamic modules

`@Module({ imports, controllers, providers, prefix })` groups related functionality. Modules can import other modules (recursive, circular-safe). `prefix` composes with controller prefixes via `joinPaths()`.

Dynamic module pattern for runtime configuration:
```ts
DrizzleModule.configure((resolve) => {
  const config = resolve(ConfigService)
  return { dialect: 'postgres', connection: { url: config.getOrThrow('DATABASE_URL') } }
})
```
`configure()` returns a `ConfiguredModule` - a module class with extra providers. `register()` adds schemas/models to an existing module's registry.

### Routing: trie-based router

`Router` uses a trie structure. Static paths: O(1) lookup. Dynamic paths with `:param` or `*wildcard`: trie traversal. HEAD falls back to GET.

HTTP method decorators: `@Get`, `@Post`, `@Put`, `@Patch`, `@Delete`, `@Head`, `@Options` - all accept optional path string.

`@Status(code)` sets default HTTP status for successful responses.

### Middleware: Koa onion model

Single `compose()` function wraps middleware array into a pipeline. Two registration points with different scopes:

- **`app.use(...middlewares)` - pre-route global.** Composed once at `compilePipelines()` into `compiledGlobalPipeline` and wraps the entire dispatch, including `router.match`. Runs on **every** request, including 404s - errors (like `NotFoundException`) bubble up through the onion, so middleware can observe them via `try { await next() } catch`. This is how CORS, request loggers, and request-id middleware see unmatched routes.
- **`@Use(...middlewares)` - route-bound.** Applies at class (all routes) or method level. Composed into per-route pipelines that run inside the inner `next()` of the global pipeline.

```ts
type Middleware = (ctx: RequestContext, next: () => Promise<void>) => void | Promise<void>
```

Semantics to remember:
- **`ctx.params` is `{}` inside global middleware before `await next()`** (router hasn't matched yet). After `await next()`, it's populated with matched route params (or stays `{}` on 404).
- **Early termination:** global middleware that does not call `next()` short-circuits dispatch - `router.match` never runs, handler never runs. The response is built from `ctx.res` (same semantics as a handler returning `null`).
- **Error observation:** a middleware wrapping `await next()` in `try/catch` sees `NotFoundException` from router, errors from per-route middleware, and errors from the handler - uniform Koa contract.

### Guards: @UseGuard and CanActivate

`@UseGuard(...guards)` applies at class or method level. `app.useGuard(...guards)` registers app-level global guards. Guards implement `CanActivate`:

```ts
interface CanActivate {
  canActivate(ctx: RequestContext): boolean | Promise<boolean>
}
```

Returns `false` → `ForbiddenException` (403). Execution order: class guards → class middleware → method guards → method middleware → handler. Global guards (from `app.useGuard()`) run first of all, but only on **matched** routes - they are not invoked on 404s.

`@SkipGuard(GuardClass)` excludes a guard from a route's pipeline at compile time. It works for **class/method-level guards AND global guards** - if the user registers `app.useGuard(AuthGuard)` and a method has `@SkipGuard(AuthGuard)`, that method bypasses the global guard entirely. Factory-wrapped guards (e.g. `AuthGuard('jwt')`) are unwrapped via the `GUARD_FACTORY` symbol, so skipping by the factory class also skips all its instances.

### Rate limiting: @miiajs/rate-limit

Fixed-window rate limiting on one core (`RateLimiter` with Upstash-style `limit(key) => { success, limit, remaining, resetMs }`). Two layers with distinct roles:

- **Guard flow (primary):** `RateLimitModule.configure({ limit, window, store?, keyGenerator?, headers? })` + `app.useGuard(RateLimitGuard)` for app-wide enforcement, `@RateLimit(policy)` for per-route/per-controller policies, `@SkipRateLimit()` to disable on a route. This is the recommended path for business rate policies.
- **Perimeter middleware:** `app.use(rateLimit({ limit, window }))` - standalone (no DI required) and the only form that covers 404s/unmatched scans (global guards never run on unmatched routes). Do NOT combine a global `rateLimit()` middleware with the decorators - the two layers know nothing about each other (`@SkipRateLimit` and replacement do not apply to middleware).

`window` accepts ms or `'500ms'`/`'10s'`/`'1m'`/`'1h'`/`'1d'` strings. On exceed both layers throw `TooManyRequestsException` (429) with `details.retryAfter`. Headers default to draft-6 `RateLimit-*` + `Retry-After`; `'legacy'` emits `X-RateLimit-*`; `false` disables (and restores the response fast path - any set header opts out of inline-JSON/LightResponse optimizations). Default key is `ctx.ip ?? 'unknown'` - behind a proxy set `trustProxy` on the app. Storage is pluggable via `RateLimitStore` (`increment()` returns hit count AND block decision atomically - contract is ready for a Redis Lua store); `MemoryStore` ships in the box, no timers, lazy expiry.

Semantics to remember:
- **Replacement, like `@BodyLimit`:** the most specific policy wins. `@RateLimit` on a method replaces the class-level policy AND the global guard for that route; on a class it replaces the global guard for all routes of the controller. A decorated route does not consume the global quota. Buckets: method = per route, class = shared by the controller's routes, global = one bucket for all matched routes; explicit `prefix` shares a bucket deliberately.
- **Skip beats specificity (unlike the `@BodyLimit` analogy):** `@SkipRateLimit()` disables rate limiting entirely on its scope - including a `@RateLimit` on the same method, and a class-level skip disables method-level `@RateLimit`s too. Use `@SkipRateLimit()`, not `@SkipGuard(RateLimitGuard)` - the latter only catches the bare global guard and explicit factory guards, not decorator-applied policies (their `GUARD_FACTORY` markers are internal scope sentinels).
- **Stacking is middleware-only:** an explicit `@UseGuard(RateLimitGuard(policy))` still stacks with the global guard, but any `@RateLimit`/`@SkipRateLimit` on that route disables it; for guaranteed stacked limits use the `rateLimit()` middleware via `@Use`.
- **Eager construction:** `app.useGuard(RateLimitGuard)` constructs the bare guard at compile time even if every route replaces it - without `RateLimitModule.configure()` the app fails at startup by design.
- **Ordering:** `RateLimitModule.configure()` must be in the same module tree (`imports`) as the controllers using `@RateLimit` - a separate later `app.register()` constructs the guards before the options provider exists.
- **`blockDuration`:** once exceeded, the key is blocked for that duration (the triggering request already gets `Retry-After` = blockDuration); after the block expires the client gets a fresh window. Optional geometric backoff (`blockBackoff` > 1 with a required `maxBlockDuration` ceiling, and `strikeReset` grace measured from the end of the block) grows the ban per repeat offence and resets strikes after a quiet period.

### Validation: @ValidateBody, @ValidateQuery, @ValidateParams

Schema-based validation via `ZodLike` interface (compatible with Zod and any schema with `safeParse()`). `@ValidateBody` internally overrides the cached body so `await ctx.json<T>()` in the handler returns validated (and possibly transformed) data. `@ValidateQuery` / `@ValidateParams` replace `ctx.query` / `ctx.params` in place. Throws `UnprocessableException` (422) with validation issues on failure.

### Request body access: ctx.json(), ctx.text()

```ts
async create(ctx: RequestContext) {
  const data = await ctx.json<CreateUserDto>()
  return this.userService.create(data)
}
```

- `ctx.json<T>()` - parses the request body as JSON and caches the result per request (Promise cache). Second call returns the same object identity.
- `ctx.text()` - parses the request body as text, cached per request.

**Single-format consumption.** Consume the body in one format per request. Calling `ctx.json()` and then `ctx.text()` (or vice versa) will throw `body already used` on Bun/Deno native runtimes and on streaming paths in node-server/uws-server. It may happen to work on node-server/uws-server optimized mode for small bodies (≤ `bufferThreshold`) thanks to an internal buffered fast path, but that's an adapter optimization detail - **do not rely on it**.

**Escape hatch.** For streaming, multipart, or binary payloads, use `ctx.req.body` (ReadableStream), `ctx.req.formData()`, or `ctx.req.arrayBuffer()` directly. These are available **only before** the first `ctx.json()` / `ctx.text()` call - once the body is consumed through the helpers, the escape hatch will throw.

### Connection info: ctx.conn, ctx.ip, trustProxy

`ctx.conn: ConnInfo` (`{ remoteAddress?, remotePort?, family? }`) is the transport-level connection info - lazy, cached per request, always the honest socket address. Sources: adapter-injected `_conn` on the Request (node-server/uws-server/TestApp) or the runtime's second fetch argument (Bun `server.requestIP()`, Deno `info.remoteAddr`); empty object when unknown (e.g. serverless). `ctx.ip` is the client IP: with `new Miia({ trustProxy })` it resolves from trusted proxy headers, otherwise equals `conn.remoteAddress`. `trustProxy` forms: `true` = leftmost `x-forwarded-for` (spoofable if the proxy appends rather than overwrites - prefer your edge's vendor header), `'cf-connecting-ip'` = single trusted header, array = priority order. `TestApp.request(method, path, { ip })` fakes the IP in tests.

### Response: ResponseBuilder

Fluent API available on `ctx.res`:
```ts
ctx.res.status(201).header('X-Custom', 'value').json({ created: true })
```

Methods: `status()`, `header()`, `json()`, `text()`, `html()`, `redirect()`, `stream()`, `build()`.

Handlers can also return plain objects (auto-serialized to JSON), `Response` instances, or `null`/`undefined` (204 No Content).

### Cookies: ctx.cookies + ResponseBuilder.cookie()

Read incoming cookies and write response cookies. `ctx.cookies` is a lazy jar; writes delegate to `ResponseBuilder`. `serializeCookie`/`parseCookieHeader`/`CookieJar`/`CookieOptions` are public.

```ts
ctx.res.cookie(name, value, options?)          // append Set-Cookie
ctx.res.deleteCookie(name, { path?, domain? }) // epoch Expires + Max-Age=0
ctx.cookies.get(name) / getAll() / has(name)   // read incoming Cookie header
ctx.cookies.set(name, value, options?) / delete(name, opts?) // delegate to res
```

Semantics to remember:
- **Defaults:** `path: '/'` defaults at `ResponseBuilder.cookie()` (NOT in `serializeCookie`); `maxAge` is in **seconds** (unlike Express ms).
- **Writes:** `cookie()`/`set()` set `_modified` and `append` (multiple `Set-Cookie` headers survive the fast path, even on error responses).
- **Secure/SameSite:** `sameSite: 'none'` auto-enables `Secure`; `secure: false` + `'none'` throws `TypeError`, as does `partitioned` without a resolved `Secure`.
- **Raw Response:** returning a native `Response` bypasses `ctx.res` - set cookies on it manually.
- **Reads:** a write is NOT visible in the same request's `get()` (jar reflects only the incoming header). Parsing: auto-`decodeURIComponent` (raw fallback), first-wins on duplicates, null-proto cache for `get()`/`has()`, `getAll()` returns a plain-object snapshot.
- **Validation:** `TypeError` on invalid RFC 6265 name, header-injection chars in `path`/`domain`, non-finite `maxAge`, invalid `expires`. `priority` is non-standard (Chromium); `partitioned` is CHIPS.

### Exceptions: HttpException hierarchy

Base `HttpException(statusCode, message, details?)` with `.toJSON()`. Derived classes:
- `BadRequestException` (400), `UnauthorizedException` (401), `ForbiddenException` (403)
- `NotFoundException` (404), `ConflictException` (409), `PayloadTooLargeException` (413)
- `UnprocessableException` (422), `TooManyRequestsException` (429), `InternalServerException` (500)

Unhandled errors in handlers are caught, logged, and returned as 500. An `Error` whose `name` is `'PayloadTooLargeError'` (thrown by node-server/uws-server body streams, which do not depend on core) is mapped to `PayloadTooLargeException` (413).

### CORS: built-in middleware

`cors(options?)` middleware in `@miiajs/core`. Options: `origin` (string/array/function), `methods`, `allowedHeaders`, `exposedHeaders`, `credentials`, `maxAge`. Handles OPTIONS preflight. Dynamic origins get `Vary: Origin` for CDN caching.

### Auth: strategy-based with DI

`@Strategy('name')` registers auth strategies as injectable providers. `AuthGuard(strategyName?)` creates a guard that resolves strategy from container and calls `strategy.validate(ctx)` to set `ctx.user`.

`@SkipGuard(AuthGuard)` (from `@miiajs/core`) excludes specific guards from a route's pipeline at compile time.

**JWT** (`@miiajs/auth/jwt`): `JwtModule.configure()` with secret/keys, `JwtService` for sign/verify via `jose`. Token extractors: `fromHeader()`, `fromCookie()`, `fromQuery()`. Abstract `JwtStrategy` with `extractToken` + `authenticate()`.

**Local** (`@miiajs/auth/local`): Abstract `LocalStrategy` with configurable `usernameField`/`passwordField` and `authenticate(username, password)`.

### Database: configure/register pattern

All DB packages (`drizzle`, `papr`, `mongoose`) follow the same pattern:
- `Module.configure(optionsOrFactory, name?)` - connection setup with retry logic
- `Module.register(models, name?)` - register models in feature modules (papr/mongoose only; drizzle is schema-first, all tables go into `configure({ schema })`)
- Models / databases become DI tokens via `defineModel(...)` (papr/mongoose) or `drizzleDb<TDb>(name?)` (drizzle). Use standard `inject(User)` / `inject(paprDb())` / `inject(mongooseConnection())` / `inject(db)`.
- Optional `name` parameter for multi-connection support
- Lifecycle: `onInit()` connects with retry, `onDestroy()` closes connection

### Swagger: OpenAPI 3.1 + Swagger UI

`SwaggerModule.configure(options)` returns a configured module - add it to your root `@Module({ imports: [...] })`. Internally, `SwaggerService` injects `DiscoveryService` + `Router` and registers the `/docs/json` + `/docs/*` endpoints from its `onReady()` hook, using `RESOLVED_PREFIX` metadata to iterate controllers without touching `ModuleLoader` internals.

Decorators: `@ApiTag`, `@ApiOperation`, `@ApiResponse`, `@ApiParam`, `@ApiQuery`, `@ApiSecurity`, `@ApiHeader`, `@ApiExclude`.

`SpecBuilder` auto-infers path params from route patterns, query params from `@ValidateQuery`, and adds 403/422 responses when guards/validation are present. Schema conversion supports Zod 3 and Zod 4.

Spec served at `{path}` (default `/docs/json`), UI at `{uiPath}/` (default `/docs/`). Only `swagger-initializer.js` is overridden.

Swagger routes register themselves with `{ skipGlobalGuards: true }` so the UI stays reachable even when the app has a global `AuthGuard` via `app.useGuard()`. Global middleware from `app.use()` (CORS, logging, request-id) still applies to swagger endpoints - only guards are opted out.

### Server: app.listen() with runtime auto-detection

`app.listen(port, hostname?, adapter?)` initializes DI, compiles routes, and starts the server. Auto-detects `Bun.serve()` / `Deno.serve()`; for Node.js/uWebSockets pass adapter: `app.listen(3000, serve)`. `destroy()` closes the server. For serverless, use `app.fetch` directly with lazy init.

**node-server / uws-server optimized mode** (default):
- Lazy Request proxy (`Object.create`, hot-path getters for method/url)
- Lightweight Headers proxy (linear scan over pairs, no `new Headers()`)
- Body buffering for small POST bodies (Content-Length ≤ `bufferThreshold`, default 100KB): `Promise<Uint8Array>` with direct `JSON.parse(textDecoder.decode(buf))`, bypasses ReadableStream + `new Request()`. Large/chunked bodies fall back to streaming.
- LightResponse cache (status/body/headers tuple, no real Response created for simple responses)
- Sync fast path (zero Promises when no middleware)

**Body size limits.** `new Miia({ maxBodySize })` (default 1MB, `false` disables) + per-route `@BodyLimit(bytes)` (method > class > app option). Limits are resolved into the route table at registration time; declared Content-Length is checked in core after route matching -> `PayloadTooLargeException` (413). Chunked bodies are capped by the adapter ceiling - `max(maxBodySize, all @BodyLimit values)` - enforced natively on Bun (`maxRequestBodySize`), via a counting stream wrapper on Deno, and by `maxBodySize` in node-server/uws-server (early 413 on Content-Length, in-stream byte cap for chunked; the stream errors with an `Error` named `'PayloadTooLargeError'` since adapters don't depend on core).

### Static file serving

`@miiajs/serve-static` exports two functions:
- `serveStatic(app, prefix, root, options?)` - registers wildcard GET route on app
- `createStaticHandler(root, options?)` - returns handler for manual registration

Features: MIME detection (50+ types, customizable), directory traversal protection, `Cache-Control`, index file with trailing slash redirect, file streaming via `node:fs`.

### Testing: TestApp + bun test

`TestApp` from `@miiajs/testing`:
```ts
const app = await TestApp.create(AppModule).override('DB', mockDb).compile()
const res = await app.request('GET', '/users')
await app.close()
```

Methods: `provide()`, `override()`, `use()`, `useGuard()`, `compile()`, `request()`, `resolve()`, `close()`.

Tests use `bun test` with explicit imports: `import { describe, it, expect } from 'bun:test'`.

### CLI: @miiajs/cli

Commands: `miia dev`, `miia build`, `miia start`, `miia check`, `miia new`, `miia generate` (alias `miia g`).

**Runtime detection:** lockfile-based (`bun.lock` → Bun, `deno.lock` → Deno, `yarn.lock`/`package-lock.json`/`pnpm-lock.yaml` → Node), fallback to executable check, override with `--runtime` flag.

**Dev mode:** runs two parallel processes — `tsc --watch` (type checking) + runtime dev server with file watching. Output from tsc piped through formatter for cleaner logs.

**Generate:** `miia g <schematic> <name>` creates individual artifacts with auto-registration in the parent `@Module`. Schematics: `module` (m), `controller` (c), `service` (s), `resource` (r), `middleware`, `guard`. The `resource` schematic creates a module + controller + service with CRUD endpoints. Flags: `--path`, `--flat`, `--dry-run`. Parent module discovery walks up from the target directory; also checks `src/app/app.module.ts` as fallback.

**New (scaffold wizard):** `miia new` runs an interactive wizard via `@clack/prompts`. Prompts: project name → runtime → package manager (pnpm/npm/yarn, skipped for Bun) → features (multiselect: Config, JWT Auth, Swagger, CORS, Serve Static) → database (single-select: Drizzle PG/MySQL/SQLite, Papr, Mongoose, None). Features that need env config auto-select Config. Scaffold generates pre-wired `app.module.ts`, `main.ts`, `.env`, and feature-specific files (auth strategies, env schema, etc.). Flags: `--dry-run`, `--skip-install`.

### Logger

`ConsoleLogger` with colored output, timestamps, PID, context tags, delta times. Configurable via `LoggerConfig`: `level` (ERROR/WARN/LOG/DEBUG), `json` mode, `appName`. Custom loggers implement `LoggerService` interface. Disable with `new Miia({ logger: false })`.

## Code conventions

- All imports use `.js` extension (ESM requirement with `verbatimModuleSyntax`)
- `sideEffects: false` in all packages
- Barrel files (`index.ts`) in each subdirectory
- `Symbol.metadata` for all decorator metadata (polyfilled via `@miiajs/core`)
- Tests in `packages/*/test/*.test.ts`, import from `bun:test`
- Biome formatting: single quotes, trailing commas, semicolons as needed, 120 char line width
- `workspace:*` for internal package dependencies
- All packages target ES2025, `module: nodenext`

## Development workflow

```sh
bun install                    # install all dependencies
bun run build                  # build all packages (tsc --build)
bun test                       # run all tests
bun run typecheck              # type-check without emitting
bun run format                 # format all files with Biome
bun run format:check           # check formatting without writing
```

Run example apps:
```sh
cd examples/drizzle-app
bun run dev                    # miia dev --runtime bun
```

---
> Source: [miiajs/miia](https://github.com/miiajs/miia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
