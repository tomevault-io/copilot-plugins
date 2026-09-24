## core-controller

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`core-controller` (package name `core-controller`, previously `@libs/core`) is a lightweight, decorator-based Express + Socket.IO framework: controllers, DI, middleware, and interceptors in the style of NestJS, built directly on `express`/`socket.io`/`reflect-metadata` rather than a full framework. It's published as a library (`dist/` is the build output referenced by `main`/`module`/`types` in package.json).

## Commands

- `npm run build:prod` — compile `src/` to `dist/` using `tsconfig.prod.json` (also runs automatically via the `prepare` script, e.g. on `npm install`/`npm publish`).
- `npm run dev` — watches `src/` and `example/` (`tsconfig.dev.json` compiles `example/` to `dev-build/`), and runs `nodemon` (see `nodemon.json`) which execs `ts-node ./example/app.ts` on change. Use this to manually exercise the framework against the example app while iterating.
- `npm test` — runs `ts-node test/simple-test.ts` directly. There is no test runner/framework (no Jest/Mocha): it's one plain script using Node's `assert`, executed top-to-bottom, testing `Service` from `example/app.ts`. There's no mechanism to run "a single test" other than editing/adding assertions in that file (or running `npx ts-node test/simple-test.ts` directly, which is equivalent to `npm test`).

There is no lint script configured.

## Architecture

The library has three cooperating layers: **decorators** (attach metadata to classes/methods via `reflect-metadata`), a **DI container** (singleton, resolves constructor/property dependencies from that metadata), and `CoreApplication` (reads the metadata at startup to wire real Express routes / Socket.IO namespaces).

### Entry point and module layout

`src/index.ts` re-exports everything consumers use: `./di`, `./enums/http-code`, `./controller`, `./core/factory`, `./http-error-exception`, `./interface`, `./type`. When adding a new public symbol, it must be exported from one of these barrel files or it won't be part of the library's public API.

- `src/di/` — `DiContainer` (singleton at `di-container.ts`) plus the `@Injectable()` and `@Inject()` decorators.
- `src/controller/` — all decorators for defining HTTP/socket controllers (`@Controller`, `@SocketController`, `@Get/@Post/@Put/@Delete/@Patch`, `@SocketEvent`, `@Param`, `@Query`, `@Body`, `@Request`, `@Response`, `@FileUpload`, socket-specific param decorators), plus `util/index.ts` (route-loading and per-request execution logic) and `constant/decorator-key.ts` (the `Symbol` keys all metadata is stored/read under).
- `src/core/factory/` — `ServerFactory.createServer()` returns a `CoreApplication` (`static-server.ts`), which does all controller registration, middleware/interceptor wiring, and server startup. `app-context.ts` and `event-bus.ts` implement a small internal pub/sub (see below).
- `src/interface/` — `Interceptor`, `ErrorInterceptor`, `CoreMiddleware`, `SocketEventAdapter`, `Action`/`Context` contracts that user code implements.
- `src/http-error-exception/` — `HttpError` (statusCode + details, plus an optional `{ bodyOnly: true }` 4th arg — see below), thrown/passed to `next()` to be caught by `ErrorInterceptor`s.
- `src/enums/http-code.ts` — `HttpStatusCode` enum.
- `example/` — a working reference app (`app.ts` + `example/controllers/**`) showing the intended usage; `tsconfig.dev.json` compiles this tree, and `nodemon`/`npm run dev` runs it. Treat this as the closest thing to living documentation/integration test for the framework.

### Decorator metadata flow

Decorators don't execute logic themselves — they call `Reflect.defineMetadata(DECORATOR_KEY.X, value, target[, propertyKey])` to tag classes/methods/params. Later, `CoreApplication` and `executeRoute` (`src/controller/util/index.ts`) read that metadata with `Reflect.getMetadata` to actually build routes and marshal arguments. All metadata keys live in `DECORATOR_KEY` (`src/controller/constant/decorator-key.ts`) — this is the map of "what a decorator writes" ↔ "what the runtime reads", and is the first place to look when tracing how a decorator affects behavior.

`@Injectable()` does one thing: registers the class with the DI `container`. It's optional on interceptors/middleware/not-found-handler classes — none of them are ever resolved through the container (`useGlobalInterceptors`/`useGlobalMiddleware`/`useNotFoundHandler` all `new` the class directly) — their role is instead declared explicitly: `@ResponseInterceptor()` for response-shaping interceptors, structural detection via `catch()` for `ErrorInterceptor`, structural detection via `use()` for `CoreMiddleware`, and simply being passed to `useNotFoundHandler()` for the not-found fallback. `@Inject()` reads the property's design-time type via `reflect-metadata` and replaces the property with a getter that resolves it from the container lazily — this works regardless of whether the *consuming* class itself is `@Injectable()`.

### Startup / request flow (`CoreApplication`, `static-server.ts`)

`ServerFactory.createServer(options)` just constructs `CoreApplication`. Controllers may be passed as classes or as glob path strings (resolved via `importClassesFromDirectories` in `controller/util/index.ts`, using `glob` + `require`). `start(port, callback)`, in order:
1. Applies request logging (`enableRequestLogging()`, dev-oriented — logs method/path/status/duration per request on response finish), then CORS and rate-limit middleware if configured.
2. Registers user middleware (`useGlobalMiddleware`) and response interceptors (`useGlobalInterceptors`).
3. `registerController()` — for each controller class: resolves constructor deps from the DI container, reads `CONTROLLER_PATH`/`CONTROLLER` metadata to distinguish plain HTTP controllers (`@Controller`) from socket controllers (`@SocketController`), and for each method reads `METHOD`/`ROUTE_PATH` metadata to either register an Express route (binding `executeRoute` as the handler, with `multer` middleware inserted first if `@FileUpload` metadata is present) or a Socket.IO event (`@SocketEvent`). Socket controllers get their own namespace (optionally suffixed by a business ID from `setBusinessId()` on the `SocketEventAdapter`), and per-event handlers marshal args from `@SocketInstance`/`@SocketCallback`/`@SocketData`/`@SocketBody` metadata, validating `@SocketBody` payloads with `class-validator`/`class-transformer` the same way HTTP bodies are.
4. Registers the not-found handler (`useNotFoundHandler`, if set) and error interceptors (`catch()`), then starts the underlying `http.Server`.

`executeRoute` (bound per-route with `{controllerInstance, methodName, appContext}`) is the actual Express handler: it reads `@Param`/`@Query`/`@Response`/`@Request`/`@FileUpload`/`@Body` metadata to build the method's argument array, validates `@Body` DTOs via `class-validator` (throwing `HttpError` → `next(err)` on failure), invokes the controller method, and if it returns a value or a Promise, routes the result through `appContext.sendJsonResponse` instead of the controller writing to `response` directly.

`AppContext` (`src/core/factory/app-context.ts`) owns a private `EventEmitter3` instance and a `WeakMap<Request, Interceptor[]>`, used to decouple response interceptors capturing request context from the final response-sending step — correlated per-request (not a shared singleton) so concurrent requests can't leak each other's interceptor. A `REQUEST_RECEIVED` event appends the firing interceptor to that request's chain; a `RESPONSE` event reduces over the chain in registration order to shape the JSON response body, then clears the entry. This is what lets `useGlobalInterceptors` transform a matched route's response shape (see `ResponseTransformerInterceptor` in the README/example, tagged `@ResponseInterceptor()`) without controllers knowing about it, and lets multiple response interceptors compose instead of only the last-registered one applying.

### Conventions for consumers (from the example app)

- Controllers use `@Inject()` for service properties (not constructor injection) and `@Controller('/path')` / `@SocketController('/path')` at the class level, `@Get/@Post/...` or `@SocketEvent` at the method level.
- Providers (services) are registered via `providers: [...]` in `ServerFactory.createServer()` and/or by being `@Injectable()`.
- DTOs use `class-validator` decorators (`@IsString()`, `@IsNumber()`, etc.); validation happens automatically for `@Body`-typed params and `@SocketBody`-typed socket payloads.
- Errors should be thrown as `HttpError(message, statusCode, details?)` and handled centrally by an `@Injectable()` class implementing `ErrorInterceptor.catch()`. `catch()`'s dispatcher (`static-server.ts`) normally maps `statusCode` straight onto the HTTP response status; pass a 4th `{ bodyOnly: true }` argument to keep a non-HTTP business/error code out of the wire status entirely — the HTTP status then falls back to `app.setDefaultErrorStatusCode()` (default 500) instead, while the code itself is still available to `ErrorInterceptor.catch()` via `error.statusCode` to put in the response body.

---
> Source: [SAS-Cambodia/core-controller](https://github.com/SAS-Cambodia/core-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
