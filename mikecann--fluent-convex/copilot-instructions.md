## fluent-convex

> Fluent builder for Convex functions with middleware. Monorepo:

# AGENTS.md

Fluent builder for Convex functions with middleware. Monorepo:
- `packages/fluent-convex` — core library (includes Zod plugin at `fluent-convex/zod`)
- `apps/example` — demo app
- `apps/docs` — interactive docs & showcase site (self-hosted on Convex via `@convex-dev/self-hosting`, auto-deployed on push to `main`)

## Testing

Always run **both** test suites after changes:

```bash
cd packages/fluent-convex && npx vitest run       # core + Zod plugin unit + type tests
cd apps/example && npx vitest run                  # integration tests (convex-test)
```

**Important:** The example app imports from the **built** `dist/` folder, not source. If you change library source files, you **must rebuild** before running downstream tests:

```bash
npm run build                                      # builds fluent-convex (includes zod subpath)
cd apps/example && npx vitest run                  # now tests see your changes
```

If you add or rename exported functions in the example app, run `cd apps/example && npx convex codegen` first.

## Middleware is onion-style

`next()` executes the rest of the chain **including the handler**. Middleware can run code before/after, catch errors, and measure timing. Do not regress to a flat loop -- see `middleware-onion.test.ts`.

## Plugin architecture (`_clone()` pattern)

The core builder uses a `protected _clone(def)` factory method. All builder methods (`.use()`, `.input()`, `.returns()`) call `this._clone()` to create new instances, so subclasses (plugins) can override `_clone()` to return their own type, preserving the plugin through the chain.

**Generic validation hooks**: `ConvexBuilderDef` has `argsTransform` and `returnsTransform` optional callback fields. `_executeWithMiddleware` calls them before/after the handler. Plugins set these hooks — e.g. `WithZod` sets `argsTransform` to `(args) => schema.parse(args)`.

**The core library's main entry has NO Zod dependency.** All Zod code lives in `src/zod/` and is exposed via the `fluent-convex/zod` subpath export. `zod` and `convex-helpers` are optional peer dependencies.

## Common pitfalls

- **Two execution paths**: `_call()` and `_register()` in `ConvexBuilderWithHandler` both use `_executeWithMiddleware`. Keep them in sync.
- **`extend()` uses prototype-based class detection** -- see `extend.ts`. Shared across all builder classes.
- **`ConvexBuilderWithHandler` constructor returns a function**, not `this`. `instanceof` will fail.
- **Plugin constructors must accept both builder and raw def** -- `_clone()` passes a raw def object, while `.extend()` passes a builder instance. Constructors should use `instanceof ConvexBuilderWithFunctionKind` to detect which.
- **Post-handler `.use()` breaks type safety** -- the handler's context type can't see middleware added after it, requiring `(context as any)` casts.
- **Circular types with `api.*` in same file** -- calling `api.myFunctions.X` in the same file without `.returns()` on the callees causes TS7022. This is a standard Convex/TS limitation, not fluent-convex specific. See commented-out example in `myFunctions.ts`.
- **Cross-function-type middleware** -- `convex.query().createMiddleware(fn)` types the input context as `QueryCtx`, so it can't be `.use()`'d on mutations or actions. For middleware that only needs shared properties (e.g. `auth`), use `convex.$context<{ auth: Auth }>().createMiddleware(fn)` to declare exactly what the middleware requires. This makes it compatible with all function types. See the README "Cross-function-type middleware" section.

---
> Source: [mikecann/fluent-convex](https://github.com/mikecann/fluent-convex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
