## learn-stateless-mcp

> enables it for localhost-class hosts — which is what you want behind a proxy; pass

# CLAUDE.md

Working guide for Claude Code in this repo. `AGENT.md` carries the same guidance for other
coding agents — **keep the two in sync when you change either.**

## What this repo is

A learning repo for **MCP protocol revision `2026-07-28`** (the stateless revision), built
twice: once with the Python SDK v2 (`mcp==2.0.0`) and once with the TypeScript SDK v2
(`@modelcontextprotocol/* @ 2.0.0`). It is teaching material — clarity beats cleverness, and
every example must actually run.

## Reference

**[REFERENCE.md](./REFERENCE.md) is the first thing to read.** It carries the `2026-07-28` wire
format (headers, `_meta` envelope, `resultType`, cache fields, the MRTR round trip), both SDK
surfaces with verified signatures, the deployment contracts, and a table mapping each failure
symptom to its cause. It was captured from the installed packages and running code, so it is
usually faster and more reliable than the spec site for anything these lessons touch. Each
lesson directory also has a README.

## Prerequisites

Node **≥ 20** (every `@modelcontextprotocol/*@2.0.0` package sets `engines.node: ">=20"`),
npm, [uv](https://docs.astral.sh/uv/), and `curl` for the `*.sh` scripts. Python does not need
installing separately — each lesson carries a `.python-version` pinning **3.13+** and
`uv sync` fetches a matching interpreter. Verified against Node 26.7, npm 11.19, uv 0.12.5.
Docker (with Compose) is needed for lessons `04-container` and `05-vercel` — verified on
Docker 29. Lessons `06-neon` and `07-rag` need a **Neon** connection string, and `07` also an
**embeddings API key**; both have free, card-free tiers, and each lesson README documents how
to obtain them. Secrets live in a git-ignored `server/.env` (copy the `.env.example` beside
it) and in the platform's env store when deploying — never in the repo. Lessons `00`–`05` need
no account of any kind. The **Vercel CLI** (`npm i -g vercel`, then `vercel login`; check with
`vercel whoami`) is needed only to *deploy* lesson `05`; both its images build and run locally
without an account. Otherwise no database, API key or network access beyond the two package registries;
lessons `00`–`03` bind `127.0.0.1` and everything runs unauthenticated.

## Commands

The two stacks are **lesson-for-lesson identical**: `00-helloworld`, `01-stateless`,
`02-pydantic`/`02-zod`, `03-mrtr`, `04-container`, `05-vercel`, `06-neon`, `07-rag`, each a
`server/` and a `client/` (except `02`, which runs both in one process; `05`–`07` add a `site/`). Both are **one project per lesson half** — a uv project in
Python, an npm package in TypeScript — installed and run where it lives. Neither `python/`
nor `typescript/` is itself a project, so there is no root install and no repo-wide test
command.

| lesson | Python (`cd python/<dir>`, `uv sync`) | TypeScript (`cd typescript/<dir>`, `npm install`) |
|---|---|---|
| 00 hello world | `00-helloworld/server` (:8000) + `00-helloworld/client` | `00-helloworld/server` (:3000) + `00-helloworld/client` |
| 01 stateless | `01-stateless/server` (:8001) + `01-stateless/client` | `01-stateless/server` (:3001) + `01-stateless/client` |
| 02 schemas | `02-pydantic` (no port) | `02-zod` (no port) |
| 03 MRTR | `03-mrtr/server` (:8003) + `03-mrtr/client` | `03-mrtr/server` (:3003) + `03-mrtr/client` |
| 04 containers | `04-container/server` (:8004) + `04-container/client` | `04-container/server` (:3004) + `04-container/client` |
| 05 Vercel | `05-vercel/server` (:8005 local, :80 in image) | `05-vercel/server` (:3005 local, :80 in image) |
| 06 Neon | `06-neon/server` (:8006) + `06-neon/client` | `06-neon/server` (:3006) + `06-neon/client` |
| 07 RAG | `07-rag/server` (:8007) + `07-rag/client` | `07-rag/server` (:3007) + `07-rag/client` |
| run | `uv run python main.py` | `npm start` |
| tests | `uv run pytest` (server projects) | `npm test` (server packages) |
| curl | `./hello.sh` / `./call.sh` / `./deploy.sh` | `npm run hello` / `call` / `deploy` |

TypeScript packages also answer to `npm run dev`, `npm run typecheck` and
`npm run build`/`npm run serve`.

Lesson `04` adds `docker build` / `docker compose up --build` (three replicas behind nginx).
Lesson `05` builds `Dockerfile.vercel` and deploys with
`vercel deploy --yes --scope <team>` **from the lesson root** (the directory holding
`vercel.json`, not `server/`).
Docker is the only extra prerequisite; the Vercel CLI is needed only to deploy or run
`vercel dev`, and neither lesson requires an account to build and run the image locally.

Servers and clients are separate processes: start the server first, in its own terminal.

## Non-negotiable rule: verify before you write

This revision and both SDK v2 lines post-date most model training data, and v2 renamed a lot.
**Do not write MCP code from memory.** Before using an API:

- introspect the installed package (`uv run python -c "import inspect; ..."`, or read
  `node_modules/@modelcontextprotocol/*/dist/*.d.mts`), or
- check the spec at <https://modelcontextprotocol.io/specification/2026-07-28>,

then **run the example** and paste real output. An example that has not been executed does not
get committed.

## Verified facts worth not rediscovering

These were each confirmed by running code in this repo.

**Python SDK v2**
- `FastMCP` is now `MCPServer` (`from mcp.server import MCPServer`). The client is
  `from mcp import Client`, used as `async with Client("http://.../mcp") as client:`.
- The client defaults to `mode="auto"`, so it negotiates `2026-07-28` on its own.
- The server picks its era from the **`MCP-Protocol-Version` header**. Send
  `MCP-Protocol-Version: 2026-07-28` with hand-rolled curl requests or you get routed to the
  2025 path and rejected with `Bad Request: Missing session ID`.
- MRTR is expressed through resolver injection: `Annotated[T, Resolve(fn)]` on a tool
  parameter, with `fn` returning `Elicit(...)`, `Sample(...)` or `ListRoots()`. The framework
  drives the round trip and the parameter never appears in the tool's input schema.
- Cache hints: `MCPServer(..., cache_hints={"tools/list": CacheHint(ttl_ms=..., scope=...)})`.
- `ctx.log()` and the Roots/Sampling surfaces raise `MCPDeprecationWarning` — deprecated in
  this revision.
- `Client(server_object)` connects **in-process** — no port, no transport — which is how
  `02-pydantic` runs a real 2026-07-28 exchange in one process. (TypeScript reaches the same
  place by injecting a `fetch` that calls `handler.fetch`.)
- `mcp.tool(...)` / `mcp.resource(...)` are decorators, but a decorator is just a callable:
  `mcp.tool(**META)(fn)` registers a function defined in another module. That is what lets the
  tools live outside the file that calls `run()`.
- The SDK derives an output schema from **any** return annotation, wrapping a scalar as
  `{"result": ...}`. Pass `structured_output=False` for tools whose answer is already a
  sentence — it is the Python way of saying "no `outputSchema`".
- Pydantic **coerces** by default, so `{"a": "21"}` reaches a `float` parameter as `21.0`;
  Zod would reject it. Ask for `strict=True` if you want the error.
- `request_state` is sealed **by default**: no `request_state_security` means
  `RequestStateSecurity.ephemeral()` — AES-256-GCM under a process-local key, so the blob
  comes back as `v1.<base64>`. That key does not survive across replicas, so any multi-replica
  deployment must pass `RequestStateSecurity(keys=[...])` (`keys[0]` seals, all keys unseal).
  The TypeScript SDK does not seal it at all — there the plaintext JSON is your problem.
- MRTR elicitation keys are **assigned by the framework** (`<module>:<resolver>`, e.g.
  `deploy:ask_for_confirmation` or `tools:ask_for_confirmation` — the prefix is the module the
  resolver lives in), not chosen by you as in TypeScript, so hand-rolled retries must read the
  key out of round one.
- **Resolvers must render a DETERMINISTIC question.** On the retry the framework re-runs the
  resolver and compares a digest of the rendered question against the one recorded when it was
  asked; if they differ it logs `Discarding the answer for resolver ...: the question changed
  since it was asked` and asks again. Interpolating a hostname, timestamp or random id into an
  `Elicit(message=...)` therefore makes every cross-replica retry loop forever — and the
  symptom is a silent re-ask, not an error. One process hides it; a fleet does not.
- `streamable_http_app()` returns a plain **Starlette** app (this is what `run()` wraps), and
  its `.routes` is an ordinary mutable list — appending a `Route('/healthz', ...)` is how you
  give an orchestrator a liveness probe, which MCP itself has no concept of.
- Behind any proxy, pass `TransportSecuritySettings(enable_dns_rebinding_protection=False)`:
  it defaults to **on** with an empty `allowed_hosts`, and the `Host` header arriving from a
  load balancer will not match, so every request is rejected.

**TypeScript SDK v2**
- The SDK is split into packages: `@modelcontextprotocol/core`, `/server`, `/client`, `/node`
  (Node adapters), `/express`, `/hono`. The old single `@modelcontextprotocol/sdk` package is
  the v1 line and stops at `1.x`. **These lessons use `/hono`, not `/express`** — see below.
- `createMcpHandler(factory, opts)` takes a **factory**, not a server instance — one fresh
  `McpServer` per request. `legacy: 'stateless' | 'reject'` decides whether 2025 traffic is
  served at all.
- **Hono is the right host, and the SDK proves it rather than taste deciding.** The core
  transport is `WebStandardStreamableHTTPServerTransport`, built on Web Standard
  `Request`/`Response`; `handler.fetch` is that shape and Hono's `c.req.raw` *is* a `Request`,
  so the integration is one line. Express speaks `IncomingMessage`/`ServerResponse` and needs a
  converter — and `@modelcontextprotocol/node` performs that conversion with
  `@hono/node-server` (it is right there in its `dependencies`, alongside a `hono` peer dep).
  Express is a shim over Hono's own adapter. Fetch-native auth (Better Auth's
  `auth.handler(Request)`) composes on the same terms.
- Hono wiring: `const app = createMcpHonoApp({ host });` then
  `app.post('/mcp', c => handler.fetch(c.req.raw, { parsedBody: c.get('parsedBody') }))`, served
  with `serve({ fetch: app.fetch, hostname, port })` from `@hono/node-server`. No adapter.
- `c.get('parsedBody')` needs `declare module 'hono' { interface ContextVariableMap {
  parsedBody?: unknown } }` — Hono's own extension point, not a cast. A direct
  `as Hono<{ Variables: … }>` does **not** typecheck against the returned `Hono<{}>`.
- **`parsedBody` is optional on Hono**, unlike Express's `req.body`. Verified both ways against
  a live server: omitting it still works, because `c.req.raw` remains consumable. Passing it
  just avoids parsing the same body twice. On Express, omitting the equivalent is fatal
  (`-32700 Parse error`) — that asymmetry is part of the argument.
- `createMcpHonoApp({ host })` installs DNS-rebinding protection automatically for
  localhost-class hosts, plus `allowedHosts`/`allowedOrigins` options for when you bind
  `0.0.0.0` behind a proxy — a superset of what the Express helper offered.
- **All five TypeScript lessons serve on Hono**, containers included. In `04`/`05`, binding
  `0.0.0.0` is what disables the localhost Host/Origin middleware — `createMcpHonoApp` only
  enables it for localhost-class hosts — which is what you want behind a proxy; pass
  `allowedHosts` to keep it on. Hono being web-standard also means those files run on
  Workers/Bun/Deno by swapping `serve()` for `export default app`. `toNodeHandler` +
  `node:http` remains the no-framework fallback, documented as such.
- The client defaults to `versionNegotiation.mode: 'legacy'`. Without
  `{ mode: 'auto' }` or `{ mode: { pin: '2026-07-28' } }` it silently talks 2025 to a
  perfectly capable 2026 server.
- MRTR server side: return `inputRequired({ inputRequests, requestState })`, and read answers
  with `acceptedContent(ctx.mcpReq.inputResponses, key, schema)`.
- MRTR client side: declare `capabilities: { elicitation: {} }` **and** register
  `client.setRequestHandler('elicitation/create', ...)` — the handler is rejected without the
  capability.
- Schemas are Zod 4, imported from the package root: `import * as z from 'zod'`. The `'zod/v4'`
  subpath in MCP docs is a Zod 3.25 transition artifact; don't reintroduce it.
- `inputSchema` takes a **schema**, not a raw shape — `inputSchema: { a: z.number() }` is the
  `@deprecated` v1-era overload that still appears in blog posts; write `z.object({ … })`.
- Prefer `.meta({ title, description, examples })` over `.describe()` — all three fields reach
  the model; examples are the cheapest accuracy available.
- `.refine()` is enforced but invisible in the published schema; `.transform()` is silently
  reduced to its input type (bare `z.toJSONSchema()` throws on transforms). Unknown arguments
  are stripped, not rejected. Say so in the tool description when it matters.
- Derive types from schemas (`z.infer`) rather than declaring them alongside; annotate
  `structuredContent` with the derived type so schema drift is a compile error. `z.input` is
  the pre-parse form (defaults optional), `z.output`/`z.infer` the post-parse form.
- The SDK publishes the **input view** of a schema. Hand-converting requires
  `z.toJSONSchema(schema, { io: 'input' })`; the default output view marks defaulted fields
  `required`, which is wrong for tool arguments.
- Cache hints belong to the **`McpServer` constructor** (`ServerOptions`), not to
  `createMcpHandler`: `new McpServer(info, { cacheHints: { 'tools/list': { ttlMs, cacheScope } } })`.
  `CreateMcpHandlerOptions` has no such field — putting it there is a TS2353 excess-property
  error, and if it is ever silenced the results ship the "do not cache" defaults
  (`ttlMs: 0`, `cacheScope: 'private'`) with no runtime complaint. The hint lands on the
  result as top-level `ttlMs`/`cacheScope`, not in `_meta`.
- `ctx.mcpReq.requestState` is an **accessor**, not a property: `<T = unknown>() => T | undefined`.
  Reading it without calling gets you a function — always truthy, so a `typeof x !== 'string'`
  guard silently takes the wrong branch. It returns the raw string you minted (no auto-parse),
  and `T` is caller-asserted with no validation, because the value came from the client.
- **Containerising the TypeScript lessons**: TypeScript 7 compiles to a platform-native binary
  pulled in as an optional dependency, so (a) `npm ci` inside a Linux image using a
  macOS-generated lockfile dies with `Unable to resolve @typescript/typescript-linux-*` — use
  `npm install` in the build stage so optional deps re-resolve for the target platform — and
  (b) it needs glibc, so build on `node:*-slim`, not Alpine/musl. `npm ci --omit=dev` is still
  fine for the runtime layer, where every dependency is pure JavaScript.
- `createMcpHandler` returns a web-standard `{ fetch, close, notify, bus }`, so `export default
  handler` is the whole deployment on Workers/Bun/Deno; `toNodeHandler` exists only for Node.
  Cloudflare's Agents SDK (`agents@0.21.0`, peer-deps `@modelcontextprotocol/server@2.0.0`)
  exports a same-named `createMcpHandler` from `agents/mcp/server` with a Workers
  `(request, env, ctx)` signature — different package, don't mix the imports.


**Vercel container images (lesson 05)** — verified against
<https://vercel.com/docs/functions/container-images> on 2026-08-21:
- The file must be named **`Dockerfile.vercel`** (or `Containerfile.vercel`); Vercel detects it
  and adds a rewrite routing traffic to the image. A plain `Dockerfile` is ignored.
- **The default port is 80**, not an injected `$PORT` — that is the opposite of Cloud Run and
  most PaaS. Override by setting `PORT` in project settings; the lesson's image sets
  `ENV PORT=80` so it needs no configuration.
- Instances **scale to zero** (5 min idle in production, 30 s in preview) and scale-down sends
  `SIGTERM` with a 30 s grace period. This makes multi-instance `requestState` key sharing
  mandatory rather than merely advisable: on Python, a per-process ephemeral key means
  tomorrow's container cannot unseal today's blob.
- Multiple services in one project (`vercel.json` → `services` with `root` + `entrypoint`,
  routed by `rewrites` whose `destination` is `{ "service": "name" }`). The docs do not state
  whether the source path survives the rewrite, so both lessons serve MCP at `/` as well as
  `/mcp` rather than depend on it.
- `vercel dev` runs the image locally but needs the Docker daemon and the Vercel CLI. Container
  Images is a permissioned feature; Secure Compute and Static IPs are not supported with it.
- **CLI usage.** Install `npm i -g vercel`; `vercel login` then `vercel whoami`. Deploy from
  the directory containing `vercel.json`. `--scope <team>` is **mandatory when
  non-interactive** and the account has several teams — the CLI refuses to guess and prints the
  commands to re-run; `vercel link` once avoids repeating it. Useful afterwards:
  `vercel logs <url> --follow`, `vercel inspect <url> --logs`, `vercel env add <NAME> production`,
  `vercel vcr ls`, `vercel list`, `vercel remove <project>`. If Deployment Protection is on,
  plain `curl` returns an auth page — use `vercel curl <path>`. In CI, set `VERCEL_TOKEN`
  rather than passing `--token`. `vercel link` writes `.vercel/` with `projectId`/`orgId`;
  it is git-ignored at the repo root and must stay that way.
- **Never commit a deployment URL** into the repo. Lesson READMEs use
  `https://<your-deployment>.vercel.app` placeholders deliberately — a personal URL in teaching
  material goes stale and ties the material to one account.
- Confirmed by deploying `typescript/05-vercel` and driving the live URL: **rewrites preserve
  the source path** (`/healthz` arrives as `/healthz`); each service is pushed as its own image
  (`vcr.vercel.com/<scope>/<project>/<service>`); the **first** deployment goes to production
  and later ones are previews unless `--prod`; a non-interactive `vercel deploy` requires an
  explicit `--scope <team>`; and the
  `Build output contains no "functions" or "static" directory` warning is expected noise for a
  container-only project. Cross-instance MRTR works in production — round one on
  `5d9e5962-914`, round two on `d60fc4b7-9d8`.


**Postgres / Neon (lesson 06)** — verified against a live Neon project:
- The lesson's thesis: **protocol** state is gone, **application** state is not. Keep the two
  words apart in any explanation; conflating them is the usual source of confusion about what
  `2026-07-28` actually removed.
- `DATABASE_URL` is the only coupling to Neon. Use the **pooled** endpoint (`-pooler` in the
  host) for the server, a **direct** one for migrations, per Neon's own guidance.
- **TLS differs by stack.** Node's `pg` verifies against Node's bundled CA list, so
  `sslmode=verify-full` works untouched. psycopg goes through libpq: it looks for
  `~/.postgresql/root.crt` and fails, and `sslrootcert=system` then fails again under
  `psycopg[binary]`, which ships its own OpenSSL with no system trust store. `db.py` appends
  `certifi.where()` when the URL names no `sslrootcert`. Never "fix" either one with
  `rejectUnauthorized: false` or `sslmode=disable`.
- **`AsyncConnectionPool` binds to its event loop** and cannot be reopened once closed. A
  module-scoped pytest fixture yields `PoolTimeout` after 30s for every test after the first;
  `PostgresStore` therefore takes its pool as a constructor argument.
- **Starlette 1.x removed `router.on_startup`.** Wrap `app.router.lifespan_context` rather than
  replacing it — the MCP app's own lifespan runs the session manager.
- The handle-based `requestState` is **TypeScript-only**, and not by oversight: the Python SDK
  owns that blob and its `RequestStateCodec` seam is synchronous by contract, so a database
  read inside `unseal` would block the loop.
- Secrets live in a git-ignored `.env` and in the platform's env store (`vercel env add`).
  Never in the repo; `.env` and `.env.*` are ignored at the root.


**pgvector / RAG (lesson 07)** — verified against Neon (pgvector 0.8.6) and Gemini embeddings:
- **HNSW refuses a `vector` wider than 2000 dimensions**: `column cannot have more than 2000
  dimensions for hnsw index`. `gemini-embedding-001` and `text-embedding-3-large` are 3072, so
  index the `halfvec` **cast** — `USING hnsw ((embedding::halfvec(3072)) halfvec_cosine_ops)` —
  which keeps full precision in storage and is HNSW-legal to 4000.
- **The query must repeat the cast** (`embedding::halfvec(n) <=> $1::halfvec(n)`) or the index
  is silently skipped and Postgres sequentially scans; results stay correct, so nothing fails
  loudly. Both suites assert `EXPLAIN` contains `Index Scan using doc_chunks_embedding_idx`.
- `<=>` is cosine **distance** (0 identical, 2 opposite); similarity is `1 - distance`, computed
  in the SELECT list — putting it in `ORDER BY` defeats the index.
- **Neither driver adapts an array to `vector`.** Build the `[0.1,0.2]` text form by hand and
  cast in SQL; `pg`/psycopg would otherwise send a Postgres array literal.
- Embeddings go over the **OpenAI `/embeddings` shape** (Gemini, OpenAI, Voyage, Mistral, vLLM
  all accept it), so the provider is `EMBEDDINGS_BASE_URL` + `EMBEDDINGS_API_KEY`. Send
  `dimensions` explicitly and assert the returned width, so a default change fails at the call
  rather than at `INSERT`.
- **Ingest must be re-runnable**: `content_hash` + `ON CONFLICT DO NOTHING` means an unchanged
  repo costs one SELECT. Both stacks chunk and hash identically, so the two indexes are
  interchangeable.
- **The MCP RAG shape**: the server retrieves and returns passages with `source`/`heading`; it
  does not call a model. The client has the model, the conversation and the intent.

## Protocol rules that examples must respect

- Never reintroduce `initialize`, `notifications/initialized` or `Mcp-Session-Id`.
- Every result carries `resultType` (`"complete"` or `"input_required"`).
- MRTR handlers must be **re-entrant**: round two is a fresh invocation with the same
  arguments plus `inputResponses`. Read the answer first; only ask when it is absent.
- `requestState` round-trips through the client and comes back attacker-controlled. Say so in
  comments wherever it appears, and never let unprotected state drive authorization.
- Prefer `subscriptions/listen` over anything session-shaped; don't use `ping`,
  `logging/setLevel`, or `notifications/roots/list_changed` — they were removed.
- Roots, Sampling and Logging are deprecated. Use them only to demonstrate deprecation.
- Authorization (not exercised by these examples, but don't contradict it in docs): client
  registration priority is pre-registered → Client ID Metadata Documents → Dynamic Client
  Registration (deprecated) → prompt the user. `iss` (RFC 9207) must be validated before
  redeeming a code, credentials are keyed by AS `issuer`, and `application_type` is required
  at DCR time.

## Conventions

- Lessons are numbered `00`–`03` and are meant to be read in order; each file's module
  docstring/header explains the concept and how to run it.
- Comments explain *why the protocol works this way*, not what the line does.
- Python: `snake_case` files, 4-space indent, and **type hints on every function** — every
  parameter and every return, in source *and* tests. On tool signatures they are load-bearing
  (the SDK derives the published schema from them) with `Annotated[T, Field(...)]` carrying
  descriptions and examples; elsewhere they are the house style. Factories returning a handler
  annotate `Callable[..., Awaitable[T]]` — `...` because the returned function has defaults and
  `Annotated[..., Resolve(...)]` metadata that a spelled-out parameter list cannot express.
  Check the whole tree with:

  ```bash
  python3 - <<'EOF'
  import ast, pathlib
  for p in sorted(pathlib.Path('python').rglob('*.py')):
      if '.venv' in p.parts or '__pycache__' in p.parts: continue
      for n in ast.walk(ast.parse(p.read_text())):
          if not isinstance(n, (ast.FunctionDef, ast.AsyncFunctionDef)): continue
          a = n.args
          miss = [x.arg for x in [*a.posonlyargs, *a.args, *a.kwonlyargs]
                  if x.annotation is None and x.arg not in ('self', 'cls')]
          if miss or n.returns is None:
              print(f"{p}:{n.lineno} {n.name}()")
  EOF
  ```
- TypeScript: `kebab-case` files, 4-space indent, single quotes, ESM with top-level `await`.
- **TypeScript typing is by inference, not by annotation** — the opposite of the Python rule
  above, and deliberately so: annotating what `tsc` already knows adds drift, not safety.
  Annotate the *boundaries* (exported handlers get `Promise<CallToolResult>`, schemas get
  `z.infer` types) and let the bodies infer. What is banned is the escape hatch: **no `any`, no
  `@ts-ignore`, no `@ts-expect-error`, no `as` on a value you have not narrowed.** Every
  tsconfig sets `strict`, plus `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` —
  all three cost nothing here and catch the two mistakes this code actually makes.
- `catch (error)` yields `unknown`. Narrow it (`error instanceof Error ? error.message :
  String(error)`); never `(error as Error).message`, which prints `undefined` for a thrown
  string at exactly the moment you are reading it to debug something. The only `as` in the
  repo outside tests is none; in tests it is `as unknown as ServerContext` for SDK stubs,
  which is honest about being a stub.
- **The two stacks mirror each other lesson for lesson** — same numbers, same tool names, same
  wire. When you change one side, change the other, and keep the comparison table in
  `python/README.md` honest. Deliberate differences (Pydantic coerces, Zod does not; Python
  seals `requestState`, TypeScript does not; `unique_words` vs `uniqueWords`) are teaching
  material: document them rather than smoothing them away.
- Python layout mirrors the TypeScript one: `<lesson>/server/main.py` is the serving half
  only, `<lesson>/server/<subject>.py` holds schemas + handlers + a `FOO_TOOL` metadata dict,
  `test_<subject>.py` runs under pytest with no server, and `<tool>.sh` curls the raw
  envelope, with a `<tool>.ps1` PowerShell twin beside it for Windows. The handlers are not an installed package, so pytest's `prepend` import mode is
  what puts each project's own directory on `sys.path` and makes `from tools import ...`
  resolve; each server project sets `testpaths = ["."]` for that reason.
- **TypeScript layout — one npm package per lesson half**, because a server and its client are
  separate programs and the packaging should say so (a client package depends on
  `@modelcontextprotocol/client` and not on `/server` or any web framework). Inside a server
  package the split is:
  - `src/index.ts` — the serving half only: transport, port, `createMcpHandler` wiring.
  - `src/<subject>.ts` — schemas, handlers and registration metadata (`FOO_TOOL`), exported.
    Name the module after what it offers (`hello.ts`, `deploy.ts`, or `tools.ts` for several).
  - `src/<subject>.test.ts` — plain `node:assert` under `tsx`, no framework, printing
    `ok — N assertions, no server started`. This is the reason for the split: `index.ts`
    listens as an import side effect, so anything defined there is reachable only over HTTP.
  - `<tool>.sh` — one curl call showing the raw 2026-07-28 envelope, wired to an npm script.
- Lessons are numbered `00`–`07` in both stacks and read in order: `00-helloworld` (one tool
  end to end), `01-stateless` (several tools, a resource, cache hints), `02-zod`/`02-pydantic`
  (the schema language itself), `03-mrtr`, `04-container` (the same server in an image, three
  replicas, proving statelessness rather than asserting it), `05-vercel` (that image on a
  platform that deletes instances between requests), `06-neon` (where the state you actually
  keep goes), `07-rag` (semantic search over that Postgres, with pgvector). Zod sits at `02` because its traps only mean something once
  you have seen a `tools/list` and an `outputSchema`, and because `03` sends a
  `requestedSchema` over the wire.
- Ports are **3000 + the lesson number** in TypeScript and **8000 + the lesson number** in
  Python, so nothing collides and both stacks run side by side. Lesson `02` needs no port in
  either language — it runs client and server in one process.
- Don't add dependencies for their own sake; the point is the SDK.

---
> Source: [panaversity/learn-stateless-mcp](https://github.com/panaversity/learn-stateless-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
