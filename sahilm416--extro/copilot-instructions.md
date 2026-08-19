## extro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

Extro is a "Next.js for Chrome extensions" framework: file-based entrypoints under `src/app/`, automatic Manifest V3 generation, and React routing — all driven by a single Vite plugin. pnpm workspace + Turborepo, ESM-only, TypeScript strict. Package manager is `pnpm@10.15.1`.

## Commands

Run from the repo root:

```bash
pnpm build          # turbo build (excludes @extrojs/docs)
pnpm build:docs     # turbo build --filter=@extrojs/docs
pnpm dev            # turbo dev (tsc -w in every package)
pnpm typecheck      # turbo typecheck
pnpm lint           # turbo lint
```

`extrojs` builds with `tsc`, plus a second `tsc -p tsconfig.runtime.json` for the content-script runtime clients (which need DOM + chrome types) — `pnpm --filter extrojs build` (or `dev` / `typecheck`). Tests use Vitest, run via `pnpm test` (turbo) or `pnpm --filter extrojs test`; specs live in `src/**/__tests__/`.

To exercise the framework end-to-end, use the example extension:

```bash
cd examples/basic
pnpm dev            # runs `extro dev` from the workspace CLI
pnpm build          # writes output/chrome-mv3-prod/
```

Load `output/chrome-mv3-dev/` (or `output/chrome-mv3-prod/`) in Chrome via "Load Unpacked". Dev and prod live in separate subdirs so dev artifacts (including the bridge-installed background SW) persist across `extro dev` sessions — no manual extension reload needed when restarting dev.

`extro dev` starts a Vite dev server for routable surfaces, writes the dev manifest + HTML shells (pointing at `http://localhost:<port>`) into `output/chrome-mv3-dev/`, and runs a build-watch sidecar for `background.js` / `content.js`. SIGINT/SIGTERM cleanly close the dev server; the on-disk dev bundle is left intact.

## Architecture

The framework is the Vite plugin (`src/plugin`). The CLI is a thin wrapper around it, and the router (`src/router`, surfaced as `extrojs/runtime`) provides the runtime that the plugin's generated code imports.

### Layout

One published package, `extrojs` (`packages/extrojs`); the runtime and the plugin are folders inside it, not separate packages (ADR 0009). Under `src/`:

- **CLI** (`index.ts` bin, `cli.ts`, `commands/`, `config.ts`, `dev-assets.ts`, `load-config.ts`, `paths.ts`, `env.ts`, `logger.ts`, `pkg.ts`) — loads `extro.config.ts` via jiti, runs `viteBuild` or `createServer` with the plugin. `config.ts` is the `.` export (`defineConfig`).
- **`plugin/`** (was `@extrojs/vite-plugin`, exposed as `extrojs/vite`) — entry detection, route scanning, manifest + HTML generation, per-surface virtual runtime modules.
- **`router/`** (was `@extrojs/router`) — `createExtroRouter`, `Link`, and the hooks. Surfaced via `extrojs/link`, `extrojs/navigation`, and internal `extrojs/runtime`.
- **`core/`** (was `@extrojs/core`) — `asset()`. No React dependency. Surfaced via `extrojs/asset`.
- **`react/`** (was `@extrojs/react`) — ambient env typing (`import.meta.env`), also shipped as `extrojs/client`.
- **`types/`** (was `@extrojs/types`) — `ManifestV3`, `ExtroConfig`, the Route manifest. Pure types.
- **`exports/`** — the thin subpath entry files (`link.ts`, `navigation.ts`, `asset.ts`, `runtime.ts`) re-exporting the curated public surface from the folders above.

### How a build is wired together

1. **AppTree scanning** (`plugin/app-tree.ts` `scanAppTree`) discovers `src/app/{popup,options,sidepanel}/page.{ts,tsx}` and `src/app/{background,content}/index.{ts,tsx}` plus nested routes. The per-routable-surface slot IS that surface's Route manifest (ADR 0007); script surfaces live under `tree.scripts`. Surfaces themselves are described by the `SURFACES` descriptor array in `plugin/surfaces.ts` (`kind: "routable" | "script"`).
2. **Rollup input rewriting** (`plugin/index.ts` `config()`): for each routable surface, the input is replaced with the virtual ID `virtual:extro/runtime/<surface>` instead of the user's `page.tsx`. Background/content stay pointing at the user's file. `entryFileNames: "[name].js"` keeps Chrome-compatible deterministic names — no hashes.
3. **Virtual modules** (`plugin/runtimes/`):
   - `virtual:extro/runtime/<surface>` (`runtime-module.ts`) — emits a tiny shim that calls `createExtroRouter(routes, { surface })`. Persists the handle on `import.meta.hot.data` so HMR doesn't re-mount via `createRoot` twice; on routes-module HMR it calls `handle.update(routes)`.
   - `virtual:extro/routes/<surface>` (`routes-module.ts`) — emits an array literal of `{ type, path, load: () => import("...") }` objects, with regex literals for dynamic routes (so the runtime can `.exec()` directly).
   - `runtimes/clients/` holds the content-script runtime clients (CSUI mount, dev bridge), compiled with the separate `tsconfig.runtime.json`.
4. **Route sort order is load-bearing** (`plugin/app-tree.ts` `sortRoutes`): statics before dynamics (so exact matches aren't shadowed), longest-first within each group, alphabetical tiebreak for filesystem-stable output. `[id]` segments become `:id` param keys with capture groups.
5. **`generateBundle()`** (`plugin/index.ts`) emits `manifest.json` (`plugin/manifest.ts`, pure — fed by the Asset inventory from `plugin/asset-inventory.ts`) and one `<surface>.html` per routable surface (`plugin/generators/html.ts`), plus icons and Public assets via `plugin/emit-assets.ts`.

### Dev mode

`packages/extrojs/src/dev-assets.ts` writes `manifest.json` and `<surface>.html` directly to `output/chrome-mv3-dev/` (bypassing Vite output) so they reference `http://localhost:<port>/<entry>` and the manifest CSP allows that origin + ws origins. Background/content scripts come from the initial `viteBuild` and are kept fresh by the build-watch sidecar in `commands/dev.ts`; a dev bridge in the background service worker applies reloads/remounts per the decisions in `plugin/dev-reactions.ts`. The CLI consumes the plugin through the `plugin/internal.ts` seam (`scanAppTree`, `discoverAssets`, `emitAssets`, dev-reaction helpers, `renderDevScreen`, `SURFACES`).

### Routing runtime

The router (`src/router`) is hash-based (`window.location.hash`). `createExtroRouter` mounts once via `createRoot`, listens for `hashchange`, matches against the routes array, lazy-imports the page module, and renders inside `RouterContext.Provider`. A `navToken` counter discards stale renders if a fast navigation lands while a previous import is still in flight. `router.replace` manually dispatches `HashChangeEvent` because `history.replaceState` alone doesn't fire it. The `Link` component renders a real `<a href="#/...">` (so middle-click still works) and prepends the `#`; with `replace` it intercepts a plain left-click and calls `router.replace`.

### Manifest generation

`packages/extrojs/src/plugin/manifest.ts`. Sources fields from `extro.config.ts` first, then `package.json`. Permissions/hostPermissions fall back to sane defaults (`storage` if background, `<all_urls>` if content) only when the user hasn't supplied them. `config.manifest` is `Object.assign`'d last as a full escape hatch.

## Conventions worth knowing

- All packages are `"type": "module"`; imports inside the source must use the `.js` extension (TS resolves it). The `tsconfig.base.json` uses `moduleResolution: "Bundler"`.
- Adding a new surface means adding a `SurfaceDescriptor` to `SURFACES` in `packages/extrojs/src/plugin/surfaces.ts` — scanning, manifest generation, and the runtime module generators all key off the descriptor (ADR 0001).
- `apps/docs` is a Fumadocs site and is excluded from the default `pnpm build` until the API stabilizes.

## Agent skills

### Issue tracker

Issues live as GitHub Issues on `Sahilm416/extro`, accessed via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Five canonical roles using their default names (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.

---
> Source: [Sahilm416/extro](https://github.com/Sahilm416/extro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
