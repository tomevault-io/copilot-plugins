## vite-plugin-vscode

> This file is a concise guide for AI coding agents (and human contributors) working on this repository. It describes what the project does, how the code is organized, which commands to run, and the important conventions and pitfalls to respect.

# Agent Guide — @tomjs/vite-plugin-vscode

This file is a concise guide for AI coding agents (and human contributors) working on this repository. It describes what the project does, how the code is organized, which commands to run, and the important conventions and pitfalls to respect.

## Project Overview

[@tomjs/vite-plugin-vscode](https://github.com/tomjs/vite-plugin-vscode) is a [Vite](https://vite.dev/) plugin for developing [VS Code extension webviews](https://code.visualstudio.com/api/references/vscode-api#WebviewPanel) with `vue`/`react` (or any Vite-supported framework). It wires the webview renderer build into the VS Code extension build:

- **Compiles the consumer's `extension` code** with Vite itself (Rolldown-based, since Vite 8) — replacing the old tsdown/tsup pipeline — with `vscode` and Node.js built-ins kept external.
- **Injects the webview renderer** into the extension at runtime: in dev it injects the `client.iife.js` script into the served HTML for `HMR`; in production it injects the final generated `index.html` (with CSP/nonce/baseUri rewriting) into the extension code via the `virtual:vscode` module.
- Supports **`esm` and `cjs`** extension output, multi-page webviews (`rollupOptions.input`), `vue`/`react` devtools injection, and an optional **electron-builder-free** packaging story.
- Ships three distributable artifacts from `src/`: the plugin itself (`index.js`), a small `getWebviewHtml` helper (`webview.js`), and the webview client shim (`client.iife.js`).

The package is published on npm as `@tomjs/vite-plugin-vscode`.

## Tech Stack

- **Language**: TypeScript (strict, `@tomjs/tsconfig` base), ESM (`"type": "module"`).
- **Build tool for this package**: [tsdown](https://tsdown.dev/) (see `tsdown.config.ts`) — **the library itself is built with tsdown**; only the _consumer's_ `extension` code compilation moved to Vite.
- **Vite**: peer `^8.0.0` (Vite 8 is Rolldown-based). The plugin invokes Vite's `build()` API to compile the consumer's extension.
- **Package manager**: [pnpm](https://pnpm.io/) (`packageManager: pnpm@10.26.2`, workspace root, `examples/*`).
- **Linting**: ESLint via `@tomjs/eslint-config` (flat config), stylelint via `@tomjs/stylelint-config` for example stylesheets.
- **Commit conventions**: `commitlint` with `@tomjs/commitlint-config`, enforced by `simple-git-hooks` + `lint-staged`. Conventional Commits style (see commit history: `feat:`, `fix:`, `chore:`, `docs:`).
- **Runtime**: Node >= 18.19; peer dependencies `@types/vscode ^1.56.0`, `vite ^8.0.0`.
- **Runtime dependencies** (externalized in the library build): `@tomjs/node`, `execa`, `lodash.merge`, `node-html-parser`, `picocolors`.

## Repository Layout

```
|-- src/                  # Plugin source (the package itself)
|  |-- index.ts           # Plugin entry: option merging + all Vite plugin hooks (serve/build)
|  |-- build.ts           # Vite build orchestration for the consumer's extension (dev watch + prod)
|  |-- types.ts           # Public option interfaces (PluginOptions, ExtensionOptions, WebviewOption)
|  |-- constants.ts       # PLUGIN_NAME, ORG_NAME, VIRTUAL_MODULE_ID, RESOLVED_VIRTUAL_MODULE_ID
|  |-- logger.ts          # Vite logger wrapper with [tomjs:vscode] prefix
|  |-- utils.ts           # Dev server URL resolution helpers
|  |-- webview/
|  |  |-- webview.ts      # getWebviewHtml helper -> dist/webview.js (+ d.ts)
|  |  |-- client.ts       # webview client shim (acquireVsCodeApi patch, commands) -> dist/client.iife.js
|  |  |-- template.html   # dev HMR webview template (iframe to VITE_DEV_SERVER_URL)
|  |  |-- global.d.ts     # ambient module declarations ('*.html')
|  |  |-- window.d.ts     # Window.acquireVsCodeApi global
|-- examples/             # Runnable demo apps: react, vue, vue-esm (ESM ext), vue-import (multi-page)
|-- env.d.ts              # Public ambient types shipped to consumers (VITE_* env vars, __getWebviewHtml__)
|-- env-webview.d.ts      # Public 'virtual:vscode' module declaration
|-- tsdown.config.ts      # Build config for publishing this package (3 targets + d.ts)
|-- tsconfig.json         # TS project config (node) + references tsconfig.web.json
|-- tsconfig.web.json     # TS config for the browser-side webview client
|-- eslint.config.mjs / commitlint.config.mjs / stylelint.config.mjs
```

## Common Commands

Run from the repository root unless noted:

| Command            | Purpose                                                                |
| ------------------ | ---------------------------------------------------------------------- |
| `pnpm install`     | Install all dependencies (workspace root + examples).                  |
| `pnpm dev`         | Watch-build the package with tsdown (`pnpm clean && tsdown --watch`).  |
| `pnpm build`       | Build the package to `dist/` (tsdown). Also runs via `prepublishOnly`. |
| `pnpm clean`       | Remove `dist/`.                                                        |
| `pnpm lint`        | Run `stylelint` then `eslint --fix` over the repo.                     |
| `pnpm lint:eslint` | ESLint with `--fix`.                                                   |

### Examples (require `pnpm build` first so `dist/` exists)

From `examples/react`, `examples/vue`, `examples/vue-esm`, or `examples/vue-import`:

| Command      | Purpose                                                                                        |
| ------------ | ---------------------------------------------------------------------------------------------- |
| `pnpm dev`   | Start Vite dev server + watch-build the extension (HMR for the webview renderer).              |
| `pnpm build` | Type-check + build (vue examples run `vue-tsc --noEmit` first) → webview + compiled extension. |

There is **no test suite**; correctness is validated through the examples and `pnpm lint` + `pnpm build`.

## How the Plugin Works

Entry point: `useVSCodePlugin(options?)` in `src/index.ts` returns **an array of two Vite plugins** — one `apply: 'serve'`, one `apply: 'build'` (enforce `post`).

### 1. Option resolution (`preMergeOptions`)

- Reads the consumer's `package.json` (throws if no `main` field) to infer output `format` (`esm` for `"type": "module"`, else `cjs`).
- Defaults: `extension.entry = 'extension/index.ts'`, `outDir = 'dist-extension'`, `target = ['node20']` (esm) / `['es2019','node14']` (cjs), `clean: true`, `treeshake: !isDev`, `external: ['vscode']`.
- Dev: `sourcemap ??= true`; prod: `minify ??= true`.
- `external` is normalized to always include `'vscode'` (handles both array and function forms).
- Since the extension build now uses Vite, **dependencies are bundled by default** — no `noExternal` handling needed.

### 2. `handleConfig` (shared by both plugins)

- When `recommended` is on, splits Vite's `build.outDir` into siblings: `dist/extension` (extension output) and `dist/webview` (renderer output), redirecting the renderer there.
- Forces single-chunk output for the webview renderer: `rolldownOptions.output.codeSplitting = false` (Rolldown) or `inlineDynamicImports = true` (Rollup), detected via `'rolldownVersion' in this.meta`.

### 3. Dev plugin (`apply: 'serve'`)

- `configResolved`: reads the built `dist/client.iife.js` and `dist/webview.js` from `__dirname` (so the package must be built first).
- `configureServer`: on the dev server listening, sets `extension.env = { NODE_ENV, VITE_DEV_SERVER_URL }` and calls `runExtensionServe` (`src/build.ts`) → Vite `build()` in **watch** mode. A per-build `writeBundle` hook fires the `onSuccess` logic. The `virtual:vscode` module is injected with `devWebviewVirtualCode`, and `watchChange` logs file changes.
- `transformIndexHtml`: injects the `client.iife.js` script into `<head>` (plus react/vue devtools `<script src="http://localhost:8097|8098">` when `devtools` is enabled).

### 4. Build plugin (`apply: 'build'`, `enforce: 'post'`)

- `transformIndexHtml`: caches each entry's rendered HTML in `prodHtmlCache` (keyed by chunk name).
- `closeBundle`: calls `genProdWebviewCode` to produce extension code containing the final HTML (with CSP injected, `{{cspSource}}`/`{{nonce}}`/`{{baseUri}}` placeholders resolved at runtime), then sets `extension.env = { NODE_ENV, VITE_WEBVIEW_DIST }` and calls `runExtensionBuild` (`src/build.ts`) → Vite `build()` (one-shot).

### 5. Extension compilation (`src/build.ts`)

- `toViteConfig` converts `ExtensionOptions` into a Vite `InlineConfig`: `configFile: false`, lib mode, `resolve.conditions: ['node']`, `env` → `define`, `external` = `vscode` + Node built-ins (+ user externals) via `getNodeExternal`, and `build.target` from the option.
- Output filename is always `[name].js` (both esm and cjs, since the consumer's package.json `type` decides the module format).
- ESM output gets a `__dirname`/`__filename` shim injected via `rolldownOptions.output.banner` — **import-free and collision-free** (uses unique aliases `__tomjs_dirname`/`__tomjs_fileURLToPath` + `import.meta.dirname` fast path; never top-level `await`, never binds common names like `path`).
- Dev (`runExtensionServe`): `build.watch` → returns a `RolldownWatcher`; the `writeBundle` plugin hook triggers the `onSuccess` handler on every build; watchers are closed on process exit.
- Prod (`runExtensionBuild`): one-shot `build()`, then runs the user `onSuccess`, logs `extension build success`.

### 6. `virtual:vscode` module

- `VIRTUAL_MODULE_ID = 'virtual:vscode'` resolves to `\0virtual:vscode` via the `@tomjs:vscode:inject` plugin; `load()` returns either the dev webview helper (`dist/webview.js` content) or the prod-generated `getWebviewHtml` code.
- Consumers `import { getWebviewHtml } from 'virtual:vscode'` in their extension to get the webview HTML (types declared in `env-webview.d.ts`).

## Packaging / Library Build (`tsdown.config.ts`)

The library is built with **tsdown** into three artifacts + d.ts:

| Entry                    | Format | Target             | Output                                  |
| ------------------------ | ------ | ------------------ | --------------------------------------- |
| `src/index.ts`           | esm    | node18.19          | `dist/index.js` + `dist/index.d.ts`     |
| `src/webview/webview.ts` | esm    | node18.19          | `dist/webview.js` + `dist/webview.d.ts` |
| `src/webview/client.ts`  | iife   | chrome89 (browser) | `dist/client.iife.js`                   |

- `external: ['vite']` (dependencies are externalized by tsdown by default), `shims: true` (tsdown injects the ESM `__dirname` shim for the plugin itself), `loader: { '.html': 'text' }` for the webview template, `fixedExtension: false` (`.js` even for esm).
- `package.json` `exports`: `.` → `./dist/index.js`, `./webview` → `./dist/webview.js`, `./client` → `./dist/client.iife.js`, `./env` → `./env.d.ts`, `./types` → `./env.d.ts`.

## Key Files at a Glance

| File                            | Responsibility                                                            |
| ------------------------------- | ------------------------------------------------------------------------- |
| `src/index.ts`                  | Public API (`useVSCodePlugin`), option merge, all Vite plugin hooks.      |
| `src/build.ts`                  | Vite builds of the consumer's extension (lib mode), dev watch, onSuccess. |
| `src/types.ts`                  | Public types: `PluginOptions`, `ExtensionOptions`, `WebviewOption`.       |
| `src/constants.ts`              | `VIRTUAL_MODULE_ID`, `RESOLVED_VIRTUAL_MODULE_ID`, names.                 |
| `src/logger.ts`                 | Vite logger with `[tomjs:vscode]` prefix and timestamps.                  |
| `src/utils.ts`                  | `resolveServerUrl`/`resolveHostname` (dev server URL).                    |
| `src/webview/webview.ts`        | `getWebviewHtml` helper shipped as `./webview`.                           |
| `src/webview/client.ts`         | Webview client shim shipped as `./client`.                                |
| `src/webview/template.html`     | Dev HMR template (iframe to the dev server).                              |
| `env.d.ts` / `env-webview.d.ts` | Consumer-facing ambient types (`VITE_*`, `virtual:vscode`).               |

## Environment Variables

Set by the plugin (via Vite `define`) and consumed in the extension/webview code:

| Variable              | When                | Meaning                                                                  |
| --------------------- | ------------------- | ------------------------------------------------------------------------ |
| `VITE_DEV_SERVER_URL` | dev (`vite serve`)  | The dev server URL (webview iframe src).                                 |
| `VITE_WEBVIEW_DIST`   | prod (`vite build`) | The webview output dir (relative), used by `getWebviewHtml`'s `baseUri`. |

`process.env.NODE_ENV` is left as a runtime reference (inherited from the Vite dev server / packaging).

## Conventions and Gotchas

- **The consumer's `package.json` drives everything** — output `format` (esm/cjs) and entry paths are inferred from it; never hardcode them.
- **`extension` code is compiled by Vite; the library itself is compiled by tsdown.** Don't reintroduce tsdown into `src/build.ts`/`src/index.ts` (runtime), and don't move the library build away from `tsdown.config.ts` without explicit instruction.
- **`virtual:vscode` injection** depends on `dist/webview.js` / `dist/client.iife.js` existing — the package must be built before running the examples (`pnpm build`).
- **ESM shim is intentionally import-free / collision-free** — it must never emit `import path from "node:path"`-style bindings (that collides with consumer code, `Identifier 'path' has already been declared`) and must never use top-level `await` (breaks `require()`-loading of the ESM plugin from a CJS `vite.config.ts`, `ERR_REQUIRE_ASYNC_MODULE`). If you touch `ESM_SHIMS`, verify both the output and that a CJS-config example still loads.
- **`handleConfig` single-chunk logic** differs by bundler: `codeSplitting: false` for Rolldown vs `inlineDynamicImports: true` for Rollup — keep both branches.
- **`__dirname` in `src/index.ts`** is used at runtime to read the built webview files; it works because tsdown's `shims: true` provides it for the library's own ESM output.
- **External modules**: `vscode` and Node built-ins must never be bundled into the extension; the consumer's own dependencies ARE bundled (self-contained extension).
- **Environment variables** are the plugin's configuration surface (`VITE_DEV_SERVER_URL`, `VITE_WEBVIEW_DIST`) — keep their typings in `env.d.ts` in sync with the `define` in `src/build.ts`.
- **Config packages**: ESLint/commitlint/stylelint use the `@tomjs/*-config` packages (not the old `@tomjs/*` names). `stylelint.config.mjs` ignores `dist`/`release`/`node_modules`.
- **Code style**: `@antfu` ESLint conventions — no semicolons, single quotes, 2-space indent, trailing commas. Run `pnpm lint` before committing.
- **Commits**: Conventional Commits (commitlint + simple-git-hooks). `CHANGELOG.md` is release-generated — don't hand-edit.
- **Documentation parity**: README has English (`README.md`) and Chinese (`README.zh_CN.md`) versions — update both when documented behavior changes.
- **Examples are the smoke tests**: after changing plugin behavior, run at least one example (`pnpm build && pnpm dev` in an example) to verify webview HMR and the extension build.

## Common Pitfalls When Editing

1. **`__dirname` in the dev `configResolved`** reads `dist/client.iife.js`/`dist/webview.js` at server startup — if these are missing/stale, the injected script is wrong. Rebuild the package after changing `src/webview/*`.
2. **Two plugin instances** (serve + build) share the `opts`/`resolvedConfig`/`prodHtmlCache` closure — keep their state consistent; the build plugin must run `enforce: 'post'` so the renderer output is complete before `closeBundle`.
3. **Env injection timing**: `extension.env` is set in `configureServer` (serve) and `closeBundle` (build) — never read `VITE_DEV_SERVER_URL`/`VITE_WEBVIEW_DIST` before those hooks set it.
4. **ESM vs CJS extension output**: same `.js` filename for both, distinguished by the consumer's `package.json` `type`. If you change `lib.fileName`, verify both a `type: module` example (`vue-esm`) and a `type: commonjs` example (`vue`) build.
5. **Watch restarts**: `runExtensionServe` keeps `RolldownWatcher`s alive; ensure they're closed on process exit (they are, via `process.once('exit')`) and don't leak on repeated `configureServer` calls.

---
> Source: [tomjs/vite-plugin-vscode](https://github.com/tomjs/vite-plugin-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
