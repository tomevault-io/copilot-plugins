## routegraph

> Fast-scan entry point for an agent modifying RouteGraph's own source. Long-form versions of everything here: [CONTRIBUTING.md](CONTRIBUTING.md) (process/rules), [ARCHITECTURE.md](ARCHITECTURE.md) (what's built), [DECISIONS.md](DECISIONS.md) (why).

# AGENTS.md — RouteGraph (contributor guide)

Fast-scan entry point for an agent modifying RouteGraph's own source. Long-form versions of everything here: [CONTRIBUTING.md](CONTRIBUTING.md) (process/rules), [ARCHITECTURE.md](ARCHITECTURE.md) (what's built), [DECISIONS.md](DECISIONS.md) (why).

> Building an app *with* RouteGraph rather than editing RouteGraph itself? You want the `AGENTS.md` that `routegraph init` scaffolds into your own project, not this one.

## Hard rules — never violate

1. `@routegraph/core` has zero runtime npm dependencies — Node built-ins only; `zod` is a type-only import.
2. No `any` — ever. Use `unknown` and narrow with guards or Zod.
3. No `require()`. ESM only, throughout.
4. `core` never imports an adapter — no Express, Fastify, Hono, Koa, or Elysia references in `core`.
5. No cross-adapter imports. Adapters are siblings, not dependents of each other.
6. `index.ts` files are re-exports only. No logic, no side effects.
7. Startup errors (`graph.load()`) are fatal — never caught and swallowed.
8. Request errors (validation failure, a thrown handler) always become an HTTP response — never crash the server.
9. No out-of-scope features (see ARCHITECTURE.md's Out of Scope section) without a new DECISIONS.md entry first.
10. New functions ship with tests. No exceptions.

## Import matrix

| Package | May import | Must NOT import |
|---|---|---|
| `core` | Node built-ins; `zod` type-only | any npm package at runtime, any adapter |
| `express` | `core`, `express` | other adapters, `client`, `docs` |
| `hono` | `core`, `hono` | other adapters, `client`, `docs` |
| `fastify` | `core`, `fastify` | other adapters |
| `elysia` | `core`, `elysia` | other adapters |
| `koa` | `core`, `koa`, `@koa/router` | other adapters |
| `watcher` | `core`, `chokidar`, `picocolors` | any framework adapter |
| `docs` | `core`, `zod` (runtime — only package allowed to touch Zod internals via `instanceof`) | any framework adapter |
| `client` | `core` (types only, no runtime import) | any adapter |
| `cli` | `core`, `docs`, `watcher`, `commander`, `picocolors`, `chokidar` | any framework adapter statically (loaded dynamically at runtime instead) |

One documented exception: `packages/core/src/zod-to-jsonschema.ts` imports `ZodTypeAny` as a type only and reads `schema._zod`/`schema._def` structurally for `toOpenAPISpec()` — erased at compile time, so `core` still has zero runtime dependency on `zod`. See DECISIONS.md #27.

## `satisfies RouteConfig`, never `as`/`: RouteConfig`

```ts
export const config = { ... } satisfies RouteConfig   // correct
export const config = { ... } as RouteConfig            // wrong — bypasses checking
export const config: RouteConfig = { ... }               // wrong — widens the type, kills inference
```

`satisfies` checks the shape *and* preserves the literal Zod schema types — that's what lets `RouteHandler<typeof config>` infer `req.params.id: string` instead of the type-erased `Record<string, string>`. Applies everywhere: every route file in `examples/`, every fixture in `packages/adapters/*/test-routes/`, and the CLI's own scaffold templates. See DECISIONS.md #16.

## Commands

```bash
pnpm install                          # link the whole workspace
pnpm build / test / typecheck / dev   # -r (recursive) across every package
pnpm --filter @routegraph/core test   # one package
pnpm --filter @routegraph/core test -- --watch
cd examples/with-express && pnpm dev  # exercise every adapter against local changes
```

## TypeScript standards

Every package's `tsconfig.json` extends the root `tsconfig.base.json` — `strict`, `exactOptionalPropertyTypes`, `noUncheckedIndexedAccess`, `noImplicitReturns`, `noFallthroughCasesInSwitch`, `NodeNext` modules/resolution. Don't loosen these per-package.

- `interface` for public API shapes, `type` for unions/utility types.
- A non-null assertion (`!`) requires a `// Safe: ...` comment stating the invariant.
- Boundary casts (dynamic imports, `Proxy` traps) need the same `// Safe: ...` comment — see `loader.ts`'s `mod.default as RouteHandler`.
- Relative imports use a `.js` extension even though the source is `.ts` (NodeNext resolution).

## Commits, PRs, changesets

- Conventional Commits: `feat(core): ...`, `fix(cli): ...`, `test(watcher): ...`, `docs(arch): ...`, `chore(deps): ...`.
- PR requires: `pnpm test` and `pnpm typecheck` both pass; ARCHITECTURE.md updated if a public API changed; DECISIONS.md updated if a non-obvious trade-off was made.
- A changeset (`pnpm changeset`) is required for every package whose published output changed — `.changeset/config.json` has `fixed: []` and `linked: []` (every package versions independently) and `access: "public"`.

## Adding a new adapter — checklist

1. `packages/adapters/<framework>/` with `src/adapter.ts` (framework req/res → `NormalizedRequest`/`NormalizedResponse`) and `src/index.ts` (the router-factory).
2. `package.json`: `@routegraph/core` as a `workspace:*` `dependency`, the framework as a `peerDependency`.
3. Implement `create<Framework>Router(graph)` — or the framework's own plugin-factory convention.
4. Map req/res carefully: param/query/header naming, whether repeated query keys collapse to arrays, imperative-send vs. returned-response (see Elysia's `ElysiaResponseCollector`).
5. Look up the route fresh per-request via `graph.getRoute(method, path)` — required for hot reload.
6. Integration tests against a `test-routes/` fixture, using real HTTP via `supertest`.
7. Add `examples/with-express/index.<framework>.ts` booting the same shared `routes/` tree.
8. Update ARCHITECTURE.md's adapter table and this file's import matrix.

## Error handling

- Typed hierarchy in `packages/core/src/errors.ts`: `ValidationError`, `RouteLoadError`, `DuplicateRouteError` — each extends `RouteGraphError`, carries `.code`.
- Never swallow an error silently — wrap and rethrow with context.
- Always `schema.safeParse(...)`, never `schema.parse(...)`, in request-validation code paths.

## Where things live

`packages/core/src/{scanner,loader,validator,RouteGraph,types,errors}.ts`; `packages/adapters/<name>/src/{adapter,index}.ts`; `packages/cli/src/{commands,utils}/`; `packages/docs/src/{middleware,exporter,route-data,schema-renderer}.ts`.

---
> Source: [udaybhangare/routegraph](https://github.com/udaybhangare/routegraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
