## reprise

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Reprise** (`symfony/reprise`) — a Symfony bundle that brings Webpack Encore's key features to modern bundlers: **Vite** and **Rsbuild/Rspack**. The bundle is a Composer package (PHP `src/`/`tests/` at the repo root); its JS side is the `@symfony/reprise` npm package, an [unplugin](https://github.com/unjs/unplugin) living under `assets/`. Full ESM, greenfield (early stage, most features are stubs).

**Design principle — do NOT re-implement what the bundler already does.** Vite and Rsbuild natively handle Sass/Less/PostCSS, TypeScript, code splitting, content hashing, source maps, minification, HMR, and the dev server. This plugin does NOT wrap or re-expose any of those (so no `enableSassLoader()`-style API from Encore). Its job is the **Symfony integration glue** that the bundlers do not provide — see below.

## Monorepo layout

The repo is a **Composer bundle** (`symfony/reprise`, PHP `src/`/`tests/` at the root) plus the **`assets/` npm package** (`@symfony/reprise`, the JS plugin and its tests), tied together by a pnpm workspace at the repo root.

## Commands

Package manager is **pnpm** (enforced via `packageManager` field). Node 22 (`.nvmrc`). The root `pnpm build`/`dev`/`test`/`lint`/`fmt` scripts run from the workspace root; `build`/`dev`/`test` delegate to the `assets` package, while `lint` (Oxlint) and `fmt` (Oxfmt) run at the root over the whole repo.

- `pnpm build` — delegates to `assets`, build via `tsdown` (bundles every `assets/src/*.ts` to `assets/dist/`)
- `pnpm dev` — delegates to `assets`, `tsdown -w`, watch/rebuild
- `pnpm lint` — `oxlint` at the root (config in `.oxlintrc.json`); `pnpm lint:fix` auto-fixes. `playground/`, `assets/test/fixtures/` and `docs/` are ignored (not library source)
- `pnpm fmt` / `pnpm fmt:check` — `oxfmt` at the root (config in `.oxfmtrc.json`); `fmt:check` is the read-only variant CI runs
- `pnpm test` — delegates to `assets`, run tests (vitest, scoped to `assets/test/` via `vitest.config.ts` so it never picks up `.references/` clones)
- `pnpm vitest run assets/test/index.test.ts` — run a single test file
- `pnpm vitest run -t "hi vitest"` — run a single test by name

### Playground (manual end-to-end verification)

`playground/` is a **full Symfony 7 PHP app** used to exercise the plugin against a real backend. It defines two entries (`app`, `admin`) and imports the plugin directly from `../assets/src` (Vite via `playground/vite.config.ts`, Rsbuild via `playground/rsbuild.config.ts`). `nodemon` rebuilds on `assets/src/**/*.ts` changes.

Run from the playground dir (the root `pnpm play` script is broken — it calls a nonexistent `dev` script):

- `npm -C playground run vite:dev` / `npm -C playground run vite:build`
- `npm -C playground run rsbuild:dev` / `npm -C playground run rsbuild:build`

## Architecture

Bundler-agnostic core + per-bundler adapters, all under `assets/src/`:

- `assets/src/core/` — pure, no bundler imports: `options.ts` (`normalizeOptions` + CDN guard + `resolvePublicPath`), `dev-server.ts` (`resolveDevOrigin`), `format.ts` (`buildEntrypoints`/`buildManifest` in the frozen v1 format), `emit.ts` (`writeSymfonyFiles`).
- `assets/src/collectors/` — turn a bundler's output into the shared `NormalizedGraph`: `vite.ts` (`bundleToGraph` from the Rollup bundle in build, `configToDevGraph` from the resolved config in serve) and `rspack.ts` (`statsToGraph` from the Rspack stats JSON).
- `assets/src/index.ts` — the `unpluginFactory` (Vite only now) + `createUnplugin` default export. Its `vite` hooks call the collectors + core: `config()` sets `base`/`outDir` and disables Vite's own manifest/publicDir copy; `generateBundle` emits the two files on build; `configureServer` writes the dev-flavoured files pointing at the dev-server origin.
- `assets/src/vite.ts` — one-line unplugin adapter `createVitePlugin(unpluginFactory)` (`@symfony/reprise/vite`).
- `assets/src/rsbuild.ts` — a **hand-written native `RsbuildPlugin`** (default export, `@symfony/reprise/rsbuild`), **not** an unplugin adapter. unplugin has no `createRsbuildPlugin`, and the Symfony integration needs Rsbuild-config-level control a raw Rspack plugin can't reach: `api.modifyRsbuildConfig` forces `tools.htmlPlugin = false` (no per-entry HTML) and disables the public-dir copy (output lives under `public/build`), and sets the output paths + dev origin; `api.onAfterCreateCompiler` taps `compiler.hooks.done` to run `statsToGraph` + core. It reuses the same core as the Vite path.
- `assets/src/types.ts` — public `Options` (`outputPath`, `publicPath`, `manifestKeyPrefix`, `devServerOrigin`) + the frozen `EntrypointsJson`/`ManifestJson`/`EntryFiles` shapes (`js`/`css`/`preload`/`dynamic`).

unplugin still earns its place for Vite and (upcoming) the Stimulus virtual module (universal `resolveId`/`load`, cross-bundler). Rspack is served exclusively through the native Rsbuild adapter — the raw Rspack unplugin adapter was dropped (Rsbuild is the supported Rspack layer).

## The Symfony integration contract (the core of this project)

Encore's real value to Symfony is two JSON files written into `outputPath`, consumed by WebpackEncoreBundle's Twig helpers (`encore_entry_script_tags()`, `encore_entry_link_tags()`, `asset()`). Generating these in Encore-compatible format is the primary work:

- **`entrypoints.json`** — maps each entry name to its asset URLs grouped by type, in load order (runtime chunks before app chunks). Optional `integrity` section for SRI hashes.
    ```json
    { "entrypoints": { "app": { "js": ["/build/runtime.js", "/build/app.js"], "css": ["/build/app.css"] } } }
    ```
- **`manifest.json`** — maps logical filename -> versioned/hashed URL, for cache-busting. Keys are prefixed with `manifestKeyPrefix` (defaults to `publicPath` minus leading slash). When `publicPath` is an absolute CDN URL (contains `://`), `manifestKeyPrefix` must be set explicitly. Reprise ports the relevant half of Encore's `validatePublicPathAndManifestKeyPrefix` (`../webpack-encore/lib/config/path-util.js`) in `normalizeOptions`: an absolute `publicPath` without an explicit `manifestKeyPrefix` throws. Encore's second branch — rejecting a `publicPath` not contained in `outputPath` — is intentionally not ported: Reprise's `outputPath` (a filesystem dir) and `publicPath` (a URL prefix) are decoupled, so that heuristic would reject valid configs. CDN URLs in `entrypoints.json`/`manifest.json` are covered end-to-end by `assets/test/integration/cdn.test.ts`.

### Dev server (build mode vs serve mode)

The plugin must behave differently depending on the bundler mode:

- **Build mode** (`vite build`, `rsbuild build`): assets are written to `outputPath` with content hashes; `entrypoints.json`/`manifest.json` point at those files under `publicPath`.
- **Serve/dev mode** (`vite`, `rsbuild dev`): the bundler's own dev server holds modules in memory and serves them over HTTP with native ESM + HMR. Here `entrypoints.json` must instead point at the dev server origin (e.g. `http://127.0.0.1:5173/build/app.js`) and inject the HMR client (`@vite/client`; React additionally needs the refresh preamble), so WebpackEncoreBundle's Twig tags load from the running dev server rather than from disk.

The dev server itself is native to Vite/Rsbuild — this plugin does not run one. Its only dev-server responsibility is detecting the mode (unplugin `meta`, or Vite's `configResolved` `command === 'serve'` vs `'build'`; Rsbuild/Rspack expose the same distinction) and emitting the dev-flavored `entrypoints.json` plus client injection. Encore's counterpart is `configureDevServerOptions()` (webpack-dev-server) in the reference `index.ts`, but that whole layer is replaced by the native dev server.

### Symfony UX / Stimulus controllers

Symfony UX ships Stimulus controllers from Composer packages, declared in `assets/controllers.json` (which controllers are enabled, `fetch: eager|lazy`, and each package's `autoimport` CSS). Local project controllers live in `assets/controllers/`.

Encore wires this with `enableStimulusBridge(controllerJsonPath)` (reference `lib/WebpackConfig.ts:882`), which only (1) adds the entries declared in `controllers.json`'s `entrypoints` map and (2) aliases `@symfony/stimulus-bridge/controllers.json` to the real file. The actual controller registration lives in the `@symfony/stimulus-bridge` npm package, whose webpack loader (`@symfony/stimulus-bridge/loader!./controllers.json`) plus a `require.context`-based lazy loader turn that JSON and the `assets/controllers/` dir into a registered Stimulus `Application`.

That loader is webpack-only, so it must be reimplemented here as a bundler-agnostic **virtual module** (unplugin `resolveId`/`load`): parse `controllers.json`, resolve each third-party controller from its npm package (honoring enabled + eager/lazy + `autoimport`), glob the local `assets/controllers/` dir, and emit the code that registers them on the Stimulus app. Prior art: `vite-plugin-symfony`'s `virtual:symfony/controllers` module.

Feature roadmap (see README): `entrypoints.json` (build + dev), `manifest.json`, asset versioning wired into the manifest, absolute/CDN `publicPath`, dev-server + HMR, SRI hashes, shared runtime chunk across entries, Symfony UX / Stimulus controllers.

## Reference: the project being replaced

The original Encore lives at `../webpack-encore`. Consult it for exact output formats and semantics:

- `index.ts` — full Encore public API (what to selectively port vs. drop as bundler-native)
- `lib/webpack/entry-points-plugin.ts` — canonical `entrypoints.json` generation logic
- `lib/plugins/manifest.ts` and `lib/utils/manifest-key-prefix-helper.ts` — `manifest.json` + key-prefix edge cases
- `test_apps/npm-with-babel/public/build/{entrypoints,manifest}.json` — real example outputs

The Encore bundle for Symfony lives at `../webpack-encore-bundle`.

## Reference: unplugin examples

Read-only clones under `.references/` (git-ignored) show how mature unplugins are built — same `index.ts` factory + thin per-bundler entry convention we use. See `reference-repos.md` for the list; each clone has an `AGENTS.md` on what to study. Most relevant: `.references/unplugin-icons` (virtual `resolveId`/`load` — the model for the Stimulus module) and `.references/unplugin-auto-import` (virtual-module code injection + `.d.ts` generation).

## Conventions

- ESM only, strict TypeScript, ES2017 target. Use the `node:` prefix for Node builtins.
- New public options go in `assets/src/types.ts` with JSDoc; keep bundler adapters trivial.
- Documentation: any user-facing feature ships with a short section in `doc/index.rst`, and that section shows **both** a Vite and an Rsbuild example (the two supported bundlers) — never document one without the other. Flip the matching `*(planned)*` marker in the feature lists (`doc/index.rst` and `README.md`) when the feature lands. Match the existing sections' natural voice; draft/polish the prose with the `natural-writing-editor` agent.
- Commit messages: Symfony style `[<Scope>] <Short description>` — PascalCase scope, imperative mood, capitalized first word, no trailing period; combine scopes as `[A][B]` when a change spans several. E.g. `[Stimulus] Emit forward-slash local controller paths`, `[Docs] Frame Stimulus usage as the Encore experience`, `[CI] Cancel superseded runs with a concurrency group`. This is the convention used across Symfony UX and WebpackEncoreBundle — **not** Conventional Commits (no `feat:`/`fix:`/`chore:` prefixes).
- Releases: the published npm package lives in `assets/` (`@symfony/reprise`); its `prepublishOnly` runs the `tsdown` build before publish.

---
> Source: [symfony/reprise](https://github.com/symfony/reprise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
