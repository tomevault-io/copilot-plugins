## cloudflare-tools

> This is `cloudflare-tools`, a monorepo with tools for developing Cloudflare Workers.

This is `cloudflare-tools`, a monorepo with tools for developing Cloudflare Workers.

## Overview

- Package manager: `bun`
- Testing framework: `vitest` (+ `@effect/vitest` in packages that use Effect)
- Linter: `oxlint`
- Formatter: `oxfmt`

Code must pass linting, formatting, and typechecking. Use `bun run check` to run all checks.

## Packages

### Published Packages

- `packages/astro`: Programmatic Astro integration implementing the framework-core `Framework` service, with the deploy target passed as a value; the wrangler-free Cloudflare target (a fork of `@astrojs/cloudflare` over `cloudflare-vite-plugin`) ships at the `./cloudflare` subpath.
- `packages/cloudflare-rolldown-plugin`: Rolldown plugin for Cloudflare Workers.
- `packages/cloudflare-runtime`: Effect-native local runtime for Cloudflare Workers, powered by `workerd`.
- `packages/cloudflare-vite-plugin`: Vite plugin for Cloudflare Workers; composes `cloudflare-rolldown-plugin` and `cloudflare-runtime`.
- `packages/framework-core`: Platform-neutral framework-integration core: the `BuildOutput` contract, the `alchemy:build-output` Vite collector plugin, the project module loader, the `Framework` service contract, and the `DeployTarget` contract (the deploy target as a value passed to framework integrations).
- `packages/nuxt`: Wrangler-free Nuxt integration implementing the `Framework` service (programmatic build over the project's `@nuxt/kit`; the Cloudflare target — nitro's `cloudflare_module` preset with `deployConfig` off and the entry/exports seam — ships at `@distilled.cloud/nuxt/cloudflare`).
- `packages/octane`: Wrangler-free OctaneJS integration implementing the `Framework` service. Octane wraps Vite, so the integration is thin: it drives the project's own `vite build` (whose Octane plugin builds client + server and whose `@octanejs/adapter-cloudflare` emits `dist/server/worker.js`) and maps the on-disk output onto `BuildOutput` — no adapter forks, no plugin injection. The Cloudflare target (adapter-contract constants) ships at `@distilled.cloud/octane/cloudflare`.
- `packages/sveltekit`: Wrangler-free SvelteKit integration implementing the `Framework` service; the Cloudflare deploy target (in-memory kit adapter + rolldown re-bundle for workerd + dev stub platform) ships at `@distilled.cloud/sveltekit/cloudflare`.
- `packages/waku`: Wrangler-free Waku integration implementing the `Framework` service (platform-neutral programmatic build/dev; the Cloudflare target — `cloudflare-vite-plugin` injection + the adapter fork — ships at `@distilled.cloud/waku/cloudflare`).

### Internals

- `upstream/workers-sdk/*`: Git submodule containing the `cloudflare/workers-sdk` repository.
- `fixtures/*`: Framework fixtures driven by the e2e harness (`e2e dev/build/preview` + Playwright smoke tests in both `live` and `dev` modes).
- `packages/tools/*`: Internal build and test utilities, including `packages/tools/e2e` (the fixture harness; target-scoped config carriage in `e2e.config.ts`).
- `packages/vendor/*`: Vendored-in packages from `cloudflare/workers-sdk`. See `packages/vendor/README.md` for more details.

## Framework Integrations

Framework packages (waku/astro/sveltekit, later nextjs) separate two concerns:

- The **framework half** — programmatic build/dev orchestration implementing framework-core's `Framework` service (`{ build, dev }` returning the `BuildOutput` contract).
- The **deploy-target half** — everything platform-specific (adapter/integration forks, bundler plugin injection, finishing passes, preview serving), passed to the framework as a `DeployTarget` value (a prop). Cloudflare is the first target; each framework ships its implementation as a subpath module (e.g. `@distilled.cloud/waku/cloudflare`). Future platforms (AWS) implement the same seams without touching framework packages or framework-core.

The precise `DeployTarget` contract, the harness config carriage, and the per-framework migration recipes live in `packages/framework-core/README.md` — read that before touching any framework package.

Doctrine for all framework/target work:

- **Wrangler-free, programmatic-only.** No `wrangler.json` is read or written anywhere; all worker configuration is in-memory options (plugin/adapter options here; Worker props on the alchemy side). We never spawn a framework's CLI binary (upstream pipelines may internally — that is upstream orchestration, not ours).
- **Platform-proxy policy.** Wherever an upstream integration uses wrangler's `getPlatformProxy` (SvelteKit `adapter-cloudflare`, OpenNext `initOpenNextCloudflareForDev`, Astro `platformProxy`), reimplement the feature in `@distilled.cloud/cloudflare-runtime` (workerd-backed Node-side proxies for `env`/`cf`/`ctx`/`caches`, configured in-memory) — never take a wrangler dependency.
- **Version pinning.** The upstream surfaces these integrations touch are `@experimental`/`unstable_`/unexported: pin exact framework versions, e2e-test against real apps in CI, and treat version bumps as deliberate migrations, not routine updates.

## File Conventions

In the `cloudflare-runtime` package:

- `.worker.ts` files represent internal Cloudflare Workers, checked against `@cloudflare/workers-types`.
- `.shared.ts` files represent code that is shared between Workers and Node.js, such as interfaces and utility functions. These cannot reference Node.js or Workers-specific APIs.
- All other `.ts` files are Node.js code, checked against `@types/node`.

Internal workers are used to implement bindings and services in `workerd`. They are imported using the `worker:` scheme. For example:

```ts
import * as MyWorker from "worker:./MyWorker.worker.ts"; // type: { modules: Record<string, string> }
```

We use `tsdown` to resolve the `worker:` imports into bundled modules that can be passed into `workerd`.

> **Important:** You must re-run `bun run build` after editing any internal worker.

## Adding a new binding type to `cloudflare-runtime`

### 1. Reference the Miniflare implementation

- In Miniflare, bindings are implemented as plugins located at `upstream/upstream/workers-sdk/packages/miniflare/src/plugins/*`.
- Each plugin defines a set of callbacks. The most important are:
  - `getBindings`: Returns a list of `Worker_Binding`s that will be added to the user's worker in `workerd`.
  - `getServices`: Returns a list of services - typically internal worker scripts - that are used to implement the binding.
  - `getExtensions`: Some simpler bindings are implemented as extensions, which are internal modules that run within the user worker, as opposed to a separate service.
- Some bindings offer local and remote implementations, and some are remote-only. Remote implementations are denoted by the `remoteProxyConnectionString` option.
- The source code for services and extensions is located in `upstream/upstream/workers-sdk/packages/miniflare/src/workers/*`.
  - Some services extend a Cloudflare internal package, such as `@cloudflare/workers-shared`.
- Look at `upstream/upstream/workers-sdk/packages/miniflare/test` to see how the binding is tested. The `cloudflare-runtime` implementation must pass all of the same test cases, to the extent possible.

### 2. Implement the remote binding (if applicable)

In `cloudflare-runtime`, create a new file in `src/bindings/*` for the binding. Remote bindings are simple to implement:

```ts
// src/bindings/KvNamespace.ts
import { makeRemoteBinding } from "../remote-bindings/RemoteBindings.ts";

export const remote = (name: string, namespaceId: string) =>
  makeRemoteBinding(
    // The binding object, passed to the Cloudflare API:
    { name, type: "kv_namespace", namespaceId, raw: true },
    // The binding implementation, which should match the `getBindings` callback from the Miniflare plugin:
    (service) => ({
      name,
      kvNamespace: service,
    }),
  );
```

### 3. Implement the local binding plugin

If the binding has a local implementation, create a directory at `src/bindings/<binding-name>` instead of a file. This should contain:

- the `<BindingName>.ts` file, which should contain the local binding plugin, a local binding hook, and the remote hook if applicable.
- worker scripts, with the `.worker.ts` extension, which are used to implement the binding.
- `index.ts`, which re-exports all files in the directory except for those ending in `.shared.ts`.

See `src/bindings/assets` and `src/bindings/rate-limit` for examples. The `cloudflare-runtime` package includes a plugin system, similar to Miniflare's but adapted for Effect.

Once a plugin is implemented, you will need to register it in `src/RuntimeServices.ts` in the `layerLocalBindings` function and the `BindingServices` type. For example:

```ts
export const layerLocalBindings = () =>
  Layer.mergeAll(
    Assets.AssetsLive,
    DevRegistryProxy.DevRegistryProxyLive,
    Hyperdrive.HyperdriveLive,
    RateLimit.RateLimitLive,
    <BindingName>.<BindingName>Live, // add new bindings to the list, in alphabetical order
  );

export type BindingServices =
  | Assets.Assets
  | DevRegistryProxy.DevRegistryProxy
  | Hyperdrive.Hyperdrive
  | RateLimit.RateLimit
  | <BindingName>.<BindingName> // add new bindings to the list, in alphabetical order
```

### 4. Adapt the relevant services and extensions

Create `.worker.ts` files in `upstream/upstream/workers-sdk/packages/miniflare/src/workers/<binding-name>` for the services and extensions that need to be adapted.

The upstream worker implementations may be more complex than we need, so you may not want to simply copy and paste. Instead, aim to match the upstream implementation as closely as possible while avoiding unnecessary abstractions and creating as few files as possible.

Some more complex workers are imported from a shared package in the `workers-sdk` monorepo. In this case, use the instructions in `packages/vendor/README.md` to vendor the package into our monorepo. One example of this is the assets binding: this uses `@cloudflare/workers-shared`, which is vendored in as `@distilled.cloud/vendor-workers-shared`.

### 5. Implement tests

Based on the tests in Miniflare, create tests in `cloudflare-runtime/test/bindings/<binding-name>`. Each test case from Miniflare should be adapted here, following the conventions of the existing tests.

---
> Source: [alchemy-run/cloudflare-tools](https://github.com/alchemy-run/cloudflare-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
