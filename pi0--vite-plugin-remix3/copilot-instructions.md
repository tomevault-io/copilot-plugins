## vite-plugin-remix3

> `vite-plugin-remix3` lets a Remix v3 app build through Vite + Nitro: client entries are bundled and hashed, SSR resolves URLs through Vite's manifest, and Nitro handles deployment. The plugin replaces what `createAssetServer` would do compile on demand in runtime.

# vite-plugin-remix3

`vite-plugin-remix3` lets a Remix v3 app build through Vite + Nitro: client entries are bundled and hashed, SSR resolves URLs through Vite's manifest, and Nitro handles deployment. The plugin replaces what `createAssetServer` would do compile on demand in runtime.

**Keep AGENTS.md updated with critical information about project.**

## Repo layout

| Path                                                       | Purpose                                                                                           |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| [src/index.ts](src/index.ts)                               | The plugin. Discovers client entries, registers `client` + `ssr` Vite environments.               |
| [build.config.ts](build.config.ts)                         | `obuild` config — bundles `src/index.ts` to `dist/index.mjs`.                                     |
| [package.json](package.json)                               | Publishes `dist/`. Peer-deps `vite ^8`. Built via `pnpm build`.                                   |
| [pnpm-workspace.yaml](pnpm-workspace.yaml)                 | Workspace root. Includes `./starter`. Overrides `vite-plugin-remix3` to `workspace:*`.            |
| [starter/](starter/)                                       | Example app consumed by `pnpm dev` / `pnpm dev:build`. Wired through the workspace override.      |
| [starter/vite.config.ts](starter/vite.config.ts)           | `plugins: [nitro(), remix()]`. No `server.ts` — Nitro auto-routes through the SSR env.            |
| [starter/app/router.ts](starter/app/router.ts)             | Route → controller mapping. Exports `router`. No default export.                                  |
| [starter/app/entry.server.ts](starter/app/entry.server.ts) | SSR entry (default `ssrEntry`). `export default { fetch: router.fetch }` — Nitro's fetch handler. |
| [starter/app/entry.client.ts](starter/app/entry.client.ts) | Client bootstrap. `import.meta.glob` map + `run({ loadModule, resolveFrame })` from `remix/ui`.   |

## Plugin shape

[src/index.ts](src/index.ts) exports a single `remix(options)` factory. Options:

- `entries` — globs (relative to root) for client entry inputs. Default `['app/entry.client.{ts,tsx,js,jsx,mjs}']`.
- `deny` — globs excluded from `entries` after expansion. Default `['app/**/*.server.*']`.
- `ssrEntry` — single SSR rollup input. Default `'app/entry.server.ts'`.

The plugin runs `enforce: 'pre'` and only implements `config()` — it returns an `environments` map registering `client` (`consumer: 'client'`, `manifest: true`, multi-input) and `ssr` (`consumer: 'server'`, single input). Entry discovery uses `node:fs/promises#glob` plus a tiny inline `globToRegex` for the deny list (no `picomatch` dep — swap in if patterns ever need brace expansion or character classes).

There is no `configureServer` middleware. Vite's default dev server serves source files at root-relative URLs; the framework's emitted URLs (`/assets/<source-path>`) are stripped of the `/assets` prefix client-side in [starter/app/entry.client.ts](starter/app/entry.client.ts) before dispatching through `import.meta.glob`. (An earlier PoC stripped this server-side via middleware; doing it in the client glob handler is enough.)

## How URLs work

In SSR-rendered HTML, scripts/styles come from `?assets=client` imports:

```ts
import entryAssets from "../entry.client.ts?assets=client";
// entryAssets.entry: "/assets/app/entry.client-<hash>.js" in prod, "/app/entry.client.ts" in dev
// entryAssets.js, .css: dependent chunks
```

Nitro's `?assets` resolver replaces the import with a manifest lookup at build time and emits `__fullstack_assets_manifest.js` next to the importer's bundle. In prod the SSR bundle inlines/imports it; the URLs hit static files Nitro serves from `.output/public/`.

The framework also emits URLs for islands (the `rmx-data` JSON) using `routes.assets.href({ path })` → `/assets/<source-path>`. In prod those resolve to manifest-mapped chunks via `import.meta.glob`; in dev they hit Vite's source pipeline directly.

## Build flow (prod)

1. Vite builds the **client** env → `.output/public/assets/<chunk>-<hash>.js` plus `.vite/manifest.json`.
2. Vite builds the **ssr** env → `node_modules/.nitro/vite/services/ssr/index.js`. Any `?assets=client` imports in this graph register their metadata.
3. Nitro fires `writeAssetsManifest()` — emits `__fullstack_assets_manifest.js` to each importer env's outDir using the data registered so far.
4. Vite builds the **nitro** env → `.output/server/index.mjs`. Nitro's auto-wiring sets the renderer to `ssrRenderer` (which dispatches to the ssr env at runtime).
5. Nitro copies public assets, writes route rules, finalizes `.output/`.

## Why an `ssr` env

Nitro's build order (see `nitro/dist/vite.mjs` `buildEnvironments`):

```text
build(client) → ... → writeAssetsManifest() → build(nitro)
```

The fullstack `?assets` plugin only populates `entry`/`js` in its manifest for the env named exactly `"client"`. And `writeAssetsManifest` runs _before_ the nitro env builds — any `?assets=client` import reachable only from the nitro graph registers its metadata too late, and the manifest comes out empty.

Workaround: register an `ssr` env (default input `app/router.ts`) so the rendering chain builds in the pre-manifest loop. Nitro auto-wires the renderer to `ssrRenderer` (from `runtime/internal/vite/ssr-renderer.mjs`), which dispatches incoming requests to that ssr env via `globalThis.__nitro_vite_envs__.ssr.fetch(req)`. The ssr env's emitted bundle is unused at runtime — the env exists so the plugin pipeline runs against the SSR import graph.

This is why the starter has no top-level `server.ts`. If a `server.ts` existed and imported `app/router.ts` directly, Nitro would treat it as the entry and pull the rendering chain (and its `?assets=client` references) into the nitro graph instead, re-introducing the broken-manifest case.

## Why `app/router.ts` is the default `ssrEntry`

[starter/app/router.ts](starter/app/router.ts) ends with `export default { fetch: router.fetch }`. That default export is what Nitro picks up as the SSR fetch handler. Registering the same file as the `ssr` env's rollup input means a single source file owns both:

- The route → handler mapping (consumed at build time by the ssr env).
- The runtime fetch entry (consumed by `ssrRenderer` at request time).

If a project splits these (e.g. an `app/entry.server.ts` re-exporting the router), point `ssrEntry` at the file that imports the rendering chain, not the bare router.

## Why a single `client` env (not one per entry)

An earlier design used one Vite env per discovered entry. Build succeeded, but `__fullstack_assets_manifest.js` had `entry: undefined` and `js: []` for every key — only an env literally named `"client"` gets URLs populated. A single `client` env with multi-input fixes this and dedupes shared chunks across entries as a bonus.

## Dev / build / publish commands

From the workspace root:

- `pnpm dev` → `cd starter && pnpm dev` (Vite dev server against the starter).
- `pnpm dev:build` → `cd starter && pnpm build && pnpm preview` (full Nitro build + preview).
- `pnpm build` → `obuild` (writes `dist/index.mjs` + `dist/index.d.mts`).
- `pnpm typecheck` → `tsgo --noEmit --skipLibCheck`.
- `pnpm lint` → `oxlint . && oxfmt --check .`. `pnpm fmt` to autofix.
- `pnpm release` → tests + build + `changelogen --release` + `npm publish` + `git push --follow-tags`.

The starter consumes the local plugin via the `vite-plugin-remix3: workspace:*` override in [pnpm-workspace.yaml](pnpm-workspace.yaml), so `pnpm dev` after `pnpm build` will use the freshly built `dist/`. If you change `src/`, rerun `pnpm build` (or run `pnpm build` once before `pnpm dev`).

## Gotchas

- **`.d.ts` files match `**/_.ts`globs.** The`import.meta.glob`in [starter/app/entry.client.ts](starter/app/entry.client.ts) explicitly excludes`!/app/\*\*/_.d.ts` to avoid Vite emitting empty 0-byte chunks for declaration files. Keep this exclusion in any new client entry.
- **`?assets=<env>` env name must be `client`** for `entry`/`js` to be populated. `?assets=app_assets_entry` (custom name) gives back only `css`. Don't rename the env without coordinating with the upstream `?assets` plugin.
- **Don't reintroduce a top-level `server.ts`** that imports `app/router.ts` directly — it pulls the rendering chain (and thus `?assets=client`) into the nitro graph, where its metadata registers too late and the resulting manifest import is unresolved at runtime.
- **Components that emit `<script src=...>` for the entry must use `entryAssets.entry`** (from `?assets=client`), not `routes.assets.href(...)` — the latter emits an unhashed source path that 404s in prod. Both [starter/app/ui/document.tsx](starter/app/ui/document.tsx) and [starter/app/ui/scaffold-home-page.tsx](starter/app/ui/scaffold-home-page.tsx) follow this pattern.
- **Plugin is config-only.** It returns `environments` from `config()` and does nothing else. Don't add hooks unless they're required — the goal is to stay a thin shim over Vite's Environments API.

## Upstream-bug-like behaviors worth filing

- `writeAssetsManifest` should run _after_ the nitro env builds, or the nitro env should build inside the pre-manifest loop. As-is, any `?assets=` import reachable only from nitro is silently dropped.
- The env-name === `"client"` check in `writeAssetsManifest` makes the `?assets=<envName>` API misleading — non-`client` envs return manifest entries with empty `entry`/`js`.

---
> Source: [pi0/vite-plugin-remix3](https://github.com/pi0/vite-plugin-remix3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
