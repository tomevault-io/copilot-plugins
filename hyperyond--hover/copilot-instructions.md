## hover

> This file is the single source of truth for agents entering the Hover repository. Read this file first. It describes the current implementation and the boundaries agents must respect when working in it.

# Directory guide

This file is the single source of truth for agents entering the Hover repository. Read this file first. It describes the current implementation and the boundaries agents must respect when working in it.

## Core documentation index

- Product scope and onboarding: `README.md`.
- Architecture and protocols: this file (`CLAUDE.md`), `packages/core/README.md`.
- License: `LICENSE` (Apache-2.0).

## What Hover is

Hover is a Vite plugin (later: a Chrome extension) that injects a floating chat widget into the user's dev server page. The developer types natural-language instructions ("test the login flow"), an agent drives their *actual* Chrome via CDP + Playwright MCP, and the verified session can be one-click crystallized into a standard Playwright `.spec.ts` file under `__vibe_tests__/`.

The differentiator vs. Stagehand / Midscene / Playwright codegen is the **AI exploration → deterministic script** workflow: AI authors the test, but the saved artifact is plain `@playwright/test` code that runs in CI without an agent in the loop.

## Workspace directories

Workspace packages come from `pnpm-workspace.yaml`: `packages/*` and `examples/*`. The repo is pnpm + ESM throughout.

- `packages/core` is `@hover-dev/core` — the Node service. Owns agent invocation, Playwright CDP preflight, MCP config, and the WebSocket bridge between the injected UI and the agent process.
- `packages/widget-bootstrap` is `@hover-dev/widget-bootstrap` — host-agnostic helper that owns the four widget source files (`template.html`, `style.css`, `client.js`, `reducer.js`), the mtime-keyed read cache, and the script-bundle assembly. Exposes three layers (`getWidgetScript` / `buildWidgetBundle` / `readWidgetAssets`) so the Vite plugin and any future bundler plugin (webpack, Next, Astro) all emit a byte-identical widget.
- `packages/transform-source` is `@hover-dev/transform-source` — **private workspace package, never published to npm** (`private: true`). Owns the per-framework source-attribution transforms that stamp `data-hover-source="<rel-path>:<line>:<col>"` onto every host element in user code: `transformJsx` (Babel parser, covers React / Solid / Preact), `transformVue` (`@vue/compiler-sfc`, filters via `tagType === ELEMENT_TYPE_HOST` so PascalCase + kebab-case components are skipped), `transformSvelte` (`svelte/compiler`'s `parse({ modern: true })`, gates on `type === 'RegularElement'` so `Component` / `SvelteHead` / `TitleElement` are skipped), and `transformAstro` (`@astrojs/compiler`, async because the underlying parser is WASM-backed; filters via `type === 'element'` so PascalCase components AND kebab-case custom-elements are both skipped). All four report the `<` character's 1-indexed line + column for cross-framework consistency. **Distributed by inlining**: each of the 5 integration shims (vite/astro/nuxt/next/webpack) runs `tsup` with `noExternal: ['@hover-dev/transform-source']`, so the transform code lands inside the shim's own `dist/` and the published bundle has no bare `@hover-dev/transform-source` import. Consumers `pnpm add` only the shim. The transform-source npm-side deps (`@babel/*`, `@vue/compiler-sfc`, `svelte`, `@astrojs/compiler`, `magic-string`) get promoted into each shim's `dependencies` so they resolve normally at install time. Package is itself `main: dist/index.js` (dist-shape) because Node's strict ESM resolver can't follow `./types.js` imports back to on-disk `.ts` — same fix used by `@hover-dev/core` and `@hover-dev/widget-bootstrap`. Root postinstall builds it on fresh clone.
- `packages/vite-plugin` is `vite-plugin-hover` — the Vite plugin shim. Consumes `@hover-dev/widget-bootstrap` for the widget injection and `@hover-dev/core` for the service. Builds with `tsup` so it can `noExternal` the private `@hover-dev/transform-source` into its dist. The plugin's `transform` hook dispatches by extension: `.jsx`/`.tsx` → JSX, `.vue` → Vue SFC, `.svelte` → Svelte 5, `.astro` → Astro. `enforce: 'pre'` is load-bearing — it puts us before `@vitejs/plugin-react` / `vue` / `svelte` transforms, which would otherwise collapse JSX/templates into render-function calls and leave no host-tag AST to walk. Must be a no-op in production builds (`apply: 'serve'`).
- `packages/astro-integration` is `@hover-dev/astro` — Astro integration shim. Astro's HTML pipeline for `.astro` pages bypasses user Vite plugins' `transformIndexHtml` output (verified empirically — `vite-plugin-hover` boots its service via `configureServer` but the widget script gets dropped), so this package wraps the same core service + widget bundle behind Astro's `injectScript('page', ...)` integration API. Source attribution is wired in two coordinated steps: (1) `updateConfig({ vite: { plugins: [...] } })` registers our `hover:source-attribution` Vite sub-plugin, but Astro's internal `astro:build` plugin claims `enforce:'pre'` first and our plugin lands AFTER it in the chain (tracked: withastro/roadmap#120); (2) `astro:server:setup` then re-sorts `server.config.plugins` to move ours to index 0. Even at index 0, **Vite's `transform()` hook still sees the already-compiled JS** because `astro:build` does the compile in its `load()` step — so for `.astro` files we intercept at `load()`: read the raw file from disk, run `transformAstro` (parse → mutate AST → serialize round-trip via `@astrojs/compiler`), and return the stamped source. Astro's load then sees our stamped output and compiles it normally. `.jsx`/`.tsx`/`.vue`/`.svelte` continue through `transform()` because they don't have an upstream `load()` step. This is unsupported maintenance-wise — Astro can break it on any minor by changing its plugin internals; the workaround is documented as a known limitation in the README. Active only on `astro dev`.
- `packages/nuxt-integration` is `@hover-dev/nuxt` — Nuxt module shim. Nuxt renders HTML through Nitro (not Vite), so Vite's `transformIndexHtml` is a no-op for Nuxt SSR/SSG responses (nuxt/nuxt#19853 — the maintainers chose this by design). This module uses `@nuxt/kit`'s `defineNuxtModule` + pushes the widget into `nuxt.options.app.head.script` (`tagPosition: 'bodyClose'`, `type: 'module'`), which Nitro renders inline into the SSR'd HTML. Source attribution is wired via `@nuxt/kit`'s `addVitePlugin()` — Nuxt's Vite chain is independent of any user-installed `vite-plugin-hover`, so the same source-attribution sub-plugin from the astro shim is registered separately here. Active only when `nuxt.options.dev === true`.
- `packages/next-integration` is `@hover-dev/next` — Next.js (App Router) integration shim. Next 16 ships Turbopack as the default bundler and Turbopack does not load webpack plugins, so `webpack-plugin-hover` only covers `next dev --webpack`; this package is the Turbopack-native path. Three pieces: (a) `withHover(nextConfig, opts)` — pure config wrapper, no side effects, serialises `opts` onto `process.env` so the runtime side can read them back; (b) `register()` from `@hover-dev/next/instrumentation` — boots the Hover service inside Next's official `instrumentation.ts` hook, which fires for `next dev` / `next start` but **not** `next build` (the reason we do NOT boot in `withHover` itself — `next.config.ts` is loaded at build time too, and a side-effect boot there would leak an orphan service into CI); (c) `<HoverScript />` — Server Component rendered after `{children}` in `app/layout.tsx`, emits a raw `<script type="module" dangerouslySetInnerHTML=…>` carrying the inlined widget bundle. Cross-file coordination uses env vars: `withHover` writes options to `__HOVER_NEXT_*`, the running service publishes its actual port to `__HOVER_NEXT_RESOLVED_PORT` after auto-bump, and `<HoverScript />` reads it back. Active only when `process.env.NODE_ENV === 'development'` AND `process.env.NEXT_RUNTIME === 'nodejs'` — the Edge runtime is unsupported because the service depends on `ws` + `cross-spawn` + `playwright-core`. **Plugin manifests (`@hover-dev/security`, future third-party plugins) go through `register()`'s second argument as a `PluginSpec[]` — either a bare module-specifier string `'@hover-dev/security'` or a `{ module, options }` object — NOT through `withHover()`'s options or via a `vite-plugin-hover`-style varargs slot on the config wrapper.** Two reasons we don't accept manifests at the config / wrapper layer: (1) `HoverOptions` is env-var-serialised across the `withHover` ↔ `register` boundary and manifests carry closures / hook functions that don't survive JSON; (2) a top-level `import securityMode from '@hover-dev/security'` in `instrumentation.ts` would be statically traced by Turbopack into the Edge bundle, pulling in mockttp / playwright-core / etc. and breaking the Edge build. Inside `register-node.ts` we resolve each specifier via the same `new Function('s','return import(s)')` opaque-import trick that `instrumentation.ts` uses to reach `register-node` itself — Turbopack can't fold a `new Function`-built specifier into its trace, so plugin packages stay Node-runtime-only. The CLI's `add` mutator does NOT auto-wire any plugin into `instrumentation.ts`; users opt in by hand-editing the `register()` call (see `examples/next-app/instrumentation.ts` for the reference shape). **Source attribution** under Next is wired via `turbopack.rules['*.{jsx,tsx}']` (also honoured by `next dev --webpack`) pointing at a `dist/source-loader.js` entry that runs `transformJsx`. We deliberately do NOT port the JSX visitor to a Rust SWC plugin: Turbopack natively accepts webpack-style loaders, the loader runs in Node at negligible cost for a one-attribute stamp, and a real SWC plugin is a 3–5 week engineering project once you account for ABI pinning + Turbopack-specific bugs. Vue/Svelte/Astro extensions are NOT routed here on purpose — Next doesn't natively handle those file shapes.
- `packages/webpack-plugin` is `webpack-plugin-hover` — webpack 5 plugin shim. Covers vanilla `webpack-dev-server`, Rspack / Rsbuild (HtmlWebpackPlugin-compatible), and legacy CRA / Vue CLI via their plugin escape hatches. **Does NOT cover Next.js by default**: Next 16+ ships Turbopack as the default bundler and Turbopack does not load webpack plugins; users have to opt into `next dev --webpack`. The Turbopack-native path is `@hover-dev/next` above. Taps `HtmlWebpackPlugin.getHooks(compilation).alterAssetTagGroups` to push a `<script type="module">` into `bodyTags`; falls back to a `processAssets` HTML splice when html-webpack-plugin isn't present. Source attribution ships as a separate `dist/loader.js` entry (exported at `./loader`); `apply()` resolves the absolute path and unshifts a `module.rules` entry with `test: /\.(jsx|tsx|vue|svelte|astro)$/` + `enforce: 'pre'` so the stamp lands before any framework compiler runs. Verified gotcha: `compiler.options.watch` is still `false` when `apply()` runs under `webpack serve` (wds flips it later), so the plugin defers service-boot to the `watchRun` compiler hook — that hook only fires in watch/serve mode, so a one-shot `webpack --mode development` build correctly doesn't spawn an orphan service.
- `packages/cli` is `@hover-dev/cli` — the zero-install setup CLI. Exposed as the `hover` bin once published. `npx @hover-dev/cli add` reads the user's `package.json` to pick the right Hover integration (Vite / Astro / Nuxt / Next / Webpack via `FRAMEWORKS` registry in `frameworks.ts`), sniffs their lockfile to pick the right package manager (pnpm / yarn / bun / npm), spawns the install command, then uses [magicast](https://github.com/unjs/magicast) to AST-mutate the user's bundler config (`vite.config.ts` / `astro.config.mjs` / `nuxt.config.ts` / `next.config.ts` / `webpack.config.js`). Supports `--vite` / `--astro` / `--nuxt` / `--next` / `--webpack` flags to force a specific framework, plus `--dry-run`. Mutators are **idempotent** — running it twice on the same project no-ops the second time. Next is the only framework whose mutator touches two files (wraps `next.config.*` in `withHover(...)` AND creates / merges an `instrumentation.ts` at the project root or under `src/`); it deliberately does NOT auto-edit `app/layout.tsx` because AST-mutating user JSX invites whitespace drift, so the `<HoverScript />` step is printed as a manual one-liner. Detection priority places `next` above `webpack` so a Next project gets routed to `@hover-dev/next` rather than the (Turbopack-incompatible) webpack plugin. Magicast gotcha: `cfg[key] ??= []` doesn't work on the proxy (the proxy returns a non-undefined value for missing keys, short-circuiting the nullish-assign and leaving subsequent pushes detached from the AST); use the explicit `ensureArray()` helper which only assigns when truly absent.
- `examples/basic-app` is the minimal Vite + React app used as the default smoke target — login + counter + todos. Vite port 5173.
- `examples/e-commerce` is an Amazon-style e-commerce SPA: product grid (with category sidebar + search) → product detail → cart → checkout (shipping address + payment method) → success. Payment method offers an inline card form OR a "Pay with PayHover" button that opens the payment-provider in a new tab and listens for the postMessage result. Stresses long action chains, cart state, conditional UI per payment method, and cross-tab popup flows. Vite port 5174.
- `examples/stock-registration` is a realistic brokerage account opening form (think IBKR / Schwab account application). 8 sections, ~50 fields, conditional reveals (foreign-tax fields when not US tax resident, previous address when current < 2 years, employer block when employed/self-employed, PEP/FINRA/control-person follow-ups, ACH bank fields when funding via ACH), multi-select chips, file upload, range slider, compliance acknowledgements. Stresses AI form filling on rich realistic-business controls. Vite port 5175.
- `examples/canvas-paint` is a drawing app: `<canvas>` for the artwork, DOM toolbar for tools/color/brush size. Stresses AI's ability to find DOM controls amidst graphical content (canvas pixels are opaque to Playwright snapshots). Vite port 5176.
- `examples/payment-provider` is a **deliberately unintegrated** mock third-party payment page used as the popup target for e-commerce's "Pay with PayHover" button. Vite port 5177. **Does NOT install `vite-plugin-hover`** — the widget must not appear on the simulated third-party origin. Stresses agent behaviour around cross-tab flows: agent must `browser_tabs(action='list')` to discover the new tab, `browser_tabs(action='select')` to switch, operate the page without a widget, and verify the original tab advances on `window.opener.postMessage` callback.
- `examples/astro-app` is the minimal Astro dogfood — single page with a counter + todo list, served via `astro dev`. Uses `@hover-dev/astro` (NOT `vite-plugin-hover`) because Astro drops user Vite plugins' `transformIndexHtml` output. Verifies that the integration-based path produces byte-equivalent widget content. Astro port 5178; service auto-bumps from 51789 like the others.
- `examples/nuxt-app` is the minimal Nuxt 4 dogfood — single `app.vue` with a counter + todo list, served via `nuxt dev`. Uses `@hover-dev/nuxt` because Nuxt's Nitro SSR pipeline bypasses Vite's `transformIndexHtml` entirely. Verifies that the module + `app.head.script` path produces byte-equivalent widget content. Nuxt port 5179; service auto-bumps from 51789. Sets `compatibilityVersion: 4` explicitly so a future Nuxt major doesn't silently shift behaviour.
- `examples/next-app` is the minimal Next.js 16 App Router dogfood — `app/page.tsx` (counter + todos, client component) + `app/layout.tsx` (Server Component embedding `<HoverScript />`) + `instrumentation.ts` (calls `registerHover()`) + `next.config.ts` (wraps export with `withHover`). Served via `next dev` (Turbopack default, no `--webpack`). Verifies the three-piece integration (config wrapper + instrumentation register + RSC script tag) produces byte-equivalent widget content to the Vite / Astro / Nuxt / Webpack outputs. Next port 5182; service auto-bumps from 51789.
- `examples/webpack-app` is the minimal vanilla webpack 5 + `webpack-dev-server` dogfood — single `src/main.js` (plain JS, no React / Babel / TypeScript on purpose) with a counter + todo list. Uses `webpack-plugin-hover` + `HtmlWebpackPlugin`. Verifies the `alterAssetTagGroups`-injection path produces byte-equivalent widget content to the Vite / Astro / Nuxt outputs. Webpack-dev-server port 5180; service auto-bumps from 51789.
- `examples/rn-web-app` is the React Native Web dogfood — Vite + React 19 + `react-native-web@0.21+`, with a one-line `react-native` → `react-native-web` Vite alias so the app's `View / Text / TextInput / Pressable` imports resolve to DOM-rendering implementations. Uses `vite-plugin-hover` (same as basic-app etc. — RN Web is just a Vite + React project at the bundler layer). Exists to make explicit what's in and out of scope: **RN Web = yes**, **RN native = no** (no DOM, no CDP, no Playwright — would be a separate product). Vite port 5181. Local `src/react-native.d.ts` is a minimal type shim covering only the components this example uses (avoids pulling in `@types/react-native` which is the native-runtime types and would diverge from RN Web's actual DOM behaviour).

Each example's Hover plugin/integration instance (`vite-plugin-hover` for the Vite examples including `rn-web-app`, `@hover-dev/astro` for `astro-app`, `@hover-dev/nuxt` for `nuxt-app`, `@hover-dev/next` for `next-app`, `webpack-plugin-hover` for `webpack-app`) starts its own Hover service. The first one to boot binds `127.0.0.1:51789`; subsequent ones auto-bump (51790, 51791, …, up to 51798). The injected widget reads `window.__HOVER_PORT__` so each example's widget connects only to its own service — running multiple examples concurrently is supported and each writes skills + specs into its own `devRoot`. `payment-provider` has no service at all.

## Inactive or placeholder directories

- `__vibe_tests__/` is the write target for crystallized Playwright specs. The directory is created by the runtime; do not hand-author placeholder files there.

## Repository status

Phase 0 (end-to-end feasibility) is verified — a `claude -p` invocation, sandboxed to only the Playwright MCP server, successfully drives the user's Chrome through a multi-step task in `examples/basic-app`. Phase 1 (Vite plugin + chat UI + persistent Node service) is the active work.

Development order is Phase 0 → 1 → 2 → 3. Phase 1 work order: WebSocket server in `@hover-dev/core` → real Vite plugin injection (`transformIndexHtml` + Shadow DOM widget) → "save as Playwright spec" file emission.

# Architecture

## Local CLI Agent First

Hover bundles no AI runtime. It spawns whatever coding-agent CLI the user already has on PATH (`claude`, `codex`, ...) and normalizes its output into a single event stream.

Supported agents today: `claude` (Claude Code, hard sandbox) and `codex` (OpenAI Codex CLI, soft sandbox). Service auto-detects the primary at startup — first installed in registry order — so a user with only `codex` installed gets Hover working without env vars. The widget shows the current agent as a pill in its header and lets the user pick another from a dropdown that also lists registered-but-not-installed agents (greyed out, with an install hint copy-pasteable from the row).

Files in `packages/core/src/agents/`:

| File | Purpose |
|---|---|
| `types.ts` | `AgentDescriptor`, `InvokeOptions`, normalized `InvokeEvent`, `SandboxStrength`, `AgentDisplay`, protocol/format enums, error classes |
| `registry.ts` | `AGENTS` constant + `listAgents()` — single source of truth for supported agents |
| `detect.ts` | `detectAgents()`, `pickPrimaryAgent()`, `listAgentAvailability()`, `resolveBinForAgent()`, `resolveOnPath()` — PATH scanning + selection |
| `argv.ts` | `buildArgv()` — protocol-aware argv construction |
| `invoke.ts` | `invokeAgent()` — async-iterable: spawn child, parse stream, yield normalized events; calls descriptor's optional `onStreamEnd` to synthesize `session_end` for agents whose protocol lacks an explicit terminator (codex) |
| `claude.ts` | Claude Code descriptor — `claude -p`, stream-json parser, hard sandbox via `--strict-mcp-config` + `--allowedTools` + `--disallowedTools` |
| `codex.ts` | OpenAI Codex descriptor — `codex exec --json`, JSONL parser, soft sandbox via `--sandbox read-only` + `developer_instructions` system prompt (codex has no built-in-tool deny list at the CLI level) |

Per-agent strategy lives in its own file. To add a new agent: write its `AgentDescriptor` and register it in `registry.ts` — nothing else changes.

`AgentDescriptor.sandboxStrength` (`'hard' | 'soft'`) is the load-bearing field that lets `service.ts` decide whether to pass the claude-style allow/disallow lists (no-op for codex, but cleaner to gate at the service layer). A `'soft'` agent gets a ⚠ badge in the widget dropdown so the user knows the built-in tool surface (`shell`, `fs_edit`, etc.) is broader than the MCP-only locked-down `'hard'` agents.

The full flow for one command: page UI → WebSocket → `@hover-dev/core` → spawn agent → MCP → Playwright → CDP → user's Chrome. Step events flow back the same path in reverse.

## Widget-driven Chrome lifecycle

The widget knows the page it's running in (`window.location.href`). The service knows which Chrome it can reach over CDP (`/json/list`). Comparing the two answers a question the user shouldn't have to: "is this widget actually in the debug Chrome?" Three answers:

| State | Meaning | Widget UI |
|---|---|---|
| `same-window` | Origin matches a CDP tab. Agent can drive this very tab. | Normal blue ✨, full UI |
| `wrong-window` | A debug Chrome exists, but this widget isn't in it. | Gray ✨, panel says "use the other window"; click → service runs `Page.bringToFront()` on the matching tab |
| `no-cdp` | No debug Chrome at all. | Amber ✨, panel says "launch debug Chrome"; click → service runs `launchDebugChrome()` |

Wire protocol additions (client → server): `check-cdp { pageUrl }`, `launch-chrome { pageUrl }`, `focus-debug { pageUrl }`. Server → client: `cdp-status { state, launching?, reason?, browser?, matchingTabUrl? }`. Widget fires `check-cdp` on every WS open (including reconnects after HMR).

Origin comparison (not full-URL) is deliberate — the user might be on `/login` while the debug Chrome tab is on `/`; they're the same app and the agent can route within it.

The Vite plugin's `autoLaunchChrome` option (default `false`) pre-warms a debug Chrome at `vite dev`. The widget's on-demand launching makes the default `false` safe — users who do nothing still get guided to a working state on first ✨ click. Examples in this monorepo opt in (`autoLaunchChrome: true`) to keep `pnpm smoke` etc. one-step.

## Boundary constraints

These are load-bearing — several are non-obvious:

- The **agent** never launches its own Chromium — it connects to whatever debug Chrome is on `chromeDebugPort` via `connectOverCDP` and picks the existing context/page whose URL matches the dev-server origin. The agent's Playwright MCP is sandboxed to a CDP target it can't change.
- The **service** is allowed to spawn one specific Chrome: the isolated debug Chrome under `<tmpdir>/hover-chrome` via `launchDebugChrome()` (in `playwright/launchChrome.ts`). This happens either at Vite startup (when `autoLaunchChrome: true`) or on widget demand (when the user clicks an amber ✨). It is *not* the user's primary Chrome profile.
- Sandboxing is per-agent. For `claude` (hard sandbox), the service passes `--strict-mcp-config`, `--permission-mode dontAsk`, `--allowedTools mcp__playwright`, `--disallowedTools "Bash Edit Write Read Grep Glob Task WebFetch WebSearch …"`. The Playwright MCP server is the only tool Claude can reach; filesystem access (other than the `__vibe_tests__/` write path) is forbidden. For `codex` (soft sandbox), there is no equivalent CLI flag to disable built-in tools — we pass `--sandbox read-only --ask-for-approval never` and inject a strict `developer_instructions` system prompt telling the agent to use only `mcp__playwright__*`. The widget marks soft-sandbox agents with a ⚠ badge so users know the surface is broader.
- Default model is `sonnet`, not `opus`. Opus is ~5× more expensive per browser-driving session. Override with `HOVER_MODEL=opus` if needed for harder tasks.
- The injected UI lives in a Shadow DOM and marks itself with `data-vibe-test="true"` so Playwright can skip it. Tailwind's default scan does not work inside Shadow DOM — use inline styles or CSS-in-JS.
- The local Node service binds to `127.0.0.1` only. The Vite plugin must be a no-op in production builds (`apply: 'serve'` in `vite-plugin-hover`).
- Generated Playwright code prefers `page.getByRole` / `page.getByText` over CSS/XPath selectors.
- Cookies / localStorage never transit the Node service; auth state stays inside the browser and is handled by Playwright in-process.
- Child-process stdio must be drained, or the spawned agent deadlocks.
- WebSocket reconnect must be robust because Vite HMR will tear down the page repeatedly during normal dev.
- Output is standard `@playwright/test` files. No proprietary test format.

## Billing risk (active)

Starting **2026-06-15**, `claude -p` calls draw from a new monthly Agent SDK credit pool separate from interactive limits. Pro: $20, Max 5x: $100, Max 20x: $200. Overage flows to API rates if usage credits are enabled, otherwise hard cutoff until refresh. Two mitigations are already in place:

1. `--max-budget-usd` ceiling per invocation (currently $0.50 in `smoke.ts`). Claude-only — codex doesn't accept this flag.
2. Local CLI Agent First — `codex` is wired (v0.3.0). Users can switch agents from the widget dropdown if `claude` becomes expensive for them, with no env-var dance. `cursor-agent`, `aider`, `gemini-cli` etc. remain one-file additions to `registry.ts`.

# Development workflow

## Environment baseline

- Runtime is Node 24+, pnpm 10+. The repo is ESM throughout. No CJS at the source layer.
- `tsconfig.base.json` at the root is the shared TS config every package extends. There is no root `tsconfig.json` — typecheck runs per-package via `pnpm typecheck`.
- Test stack: Vitest for unit tests (per-package, under `packages/*/tests/`), Playwright dogfooding for integration (crystallized specs under `examples/basic-app/__vibe_tests__/`). No linter or formatter is configured yet.

## Package entry-point conventions (Next-tax)

Most Hover packages set `main` / `exports` directly to `src/*.ts`, so consumers' transpilers (Vite, esbuild via Astro / Nuxt / Webpack, tsx for `pnpm smoke`) see TypeScript source with zero build step — the dev loop is "edit `.ts` → HMR." This works because every consumer pipeline in the repo *transpiles* what it imports.

**Exceptions: `@hover-dev/core` and `@hover-dev/widget-bootstrap`** point `main` / `exports` at `dist/*.js`, and ship a `dev: tsc --watch` script. Reason: `@hover-dev/next` consumes them via `await import(...)` from inside Next's `instrumentation.ts`, and Next 16's Turbopack resolver does not rewrite NodeNext-style `.js` import specifiers back to the on-disk `.ts` files inside transitively-traced source packages (open issue [vercel/next.js#82945](https://github.com/vercel/next.js/issues/82945) — `webpack`'s `resolve.extensionAlias` has no Turbopack equivalent). Switching those two packages to `dist`-entry shape is what lets a fresh-clone monorepo dev loop work for `examples/next-app` under default Turbopack.

**Practical implications:**

- A root `postinstall` hook (in the top-level `package.json`) runs `pnpm --filter @hover-dev/core --filter @hover-dev/widget-bootstrap build` after every `pnpm install`. Fresh clones get usable `dist/` artefacts before anyone touches an example.
- `pnpm dev:example:next-app` spawns `concurrently` with three watchers in parallel: `tsc --watch` for core, `tsc --watch` for widget-bootstrap, and `next dev` itself. Edits to `packages/core/src/service.ts` re-emit `packages/core/dist/service.js` in ~500 ms, Next picks up the changed `dist` file and HMRs the page. Cold start (initial build + Next boot) is ~5 s.
- Other examples (Vite / Astro / Nuxt / Webpack / RN Web) still tolerate `src`-entry shape — their bundlers transpile on the fly — but they now consume those two packages via `dist` too. The `postinstall` guarantees `dist` exists; the watchers aren't running for those examples, so an edit to `core/src/` requires either a one-shot `pnpm --filter @hover-dev/core build` or a separate `pnpm --filter @hover-dev/core dev` terminal. For now this is acceptable since cross-package edits during example dev are rare.
- **Sunset plan:** once vercel/next.js#82945 ships, we should switch `@hover-dev/core` and `@hover-dev/widget-bootstrap` back to `src`-entry shape and delete the watcher dance from `dev:example:next-app`. Tracking comment lives in `packages/next-integration/src/withHover.ts`.

## Edge runtime isolation in `@hover-dev/next`

Next 16 compiles `instrumentation.ts` for *both* the Node.js and the Edge runtime, and statically traces every `import` (including `await import('...')` with a literal-string specifier) into the Edge bundle even when a `process.env.NEXT_RUNTIME === 'nodejs'` runtime guard would skip the code path. The Edge bundler then chokes on Node-only transitive deps — most notably `playwright-core`'s CJS `require('chromium-bidi/...')` and our `process.cwd` / `process.once` calls.

Two structural tricks defuse this:

1. **Split file.** Everything Node-shaped lives in `packages/next-integration/src/register-node.ts`; `instrumentation.ts` is a thin shell that does the runtime guard and forwards to `registerNode()`. Edge bundling stops at the `register-node.ts` boundary if we can keep its import statically opaque.
2. **String-variable indirection** in `instrumentation.ts`: `const specifier = './register-node.js'; await import(specifier)`. Turbopack's static tracer gives up on dynamic specifiers it can't fold to a literal, so `register-node.ts` (and its transitive `@hover-dev/core/service` chain) is left out of the Edge bundle entirely. The Node runtime resolves the variable at execution time and loads the full file.

If you ever need to add another `import` from inside `register-node.ts`, no special handling is required — the whole file is already on the Node-only side of the indirection.

## Local lifecycle

Two terminals on first run; once Chrome and Vite are up they stay running across many smoke loops:

1. `pnpm dev:example:basic-app` — basic-app at http://localhost:5173. Because the example sets `autoLaunchChrome: true`, this also spawns the debug Chrome (`--remote-debugging-port=9222`, isolated profile at `<tmpdir>/hover-chrome`) navigated to the dev URL. (Same for `dev:example:e-commerce` / `…:stock-registration` / `…:canvas-paint` on 5174 / 5175 / 5176.)
2. `pnpm smoke` — end-to-end: detect agents → CDP preflight → invoke `claude` → stream events.

Need the debug Chrome without a Vite example? `pnpm smoke:chrome` standalone-spawns it (same `<tmpdir>/hover-chrome` profile, idempotent).

Custom target / prompt:

```bash
pnpm smoke http://localhost:5173/ "log in, then add a todo named 'verify hover'"
```

Environment overrides:

```bash
HOVER_AGENT=claude HOVER_MODEL=sonnet HOVER_CDP=http://localhost:9222 pnpm smoke
```

## Git commit policy

- Use Conventional Commits. Format: `<type>(<scope>): <description>`.
- Common types: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `perf`.
- `<scope>` is the package or sub-area: `core`, `vite-plugin`, `example`, `agents`, `playwright`, `mcp`, `ci`, `deps`.
- Commit messages are written in **English**. Hover is a public open-source repo; contributors come from everywhere.
- The subject line is imperative, ≤72 characters, no trailing period.
- Conventional Commits is enforced at commit time by a husky `commit-msg` hook running `commitlint`. The hook installs on `pnpm install`. Do not bypass it with `--no-verify` unless you have explicit owner sign-off.
- Stage files explicitly by name. Do not use `git add -A` / `git add .` — this avoids accidentally committing secrets or large binaries.
- Never amend a previous commit unless explicitly requested. Create a new commit instead.
- Never force-push to `main`. Never skip hooks. Never modify git config.

## Branching policy

- `main` must stay runnable. Every commit pushed to `main` should, in theory, leave the basic flow (`pnpm install` → `pnpm typecheck` → `pnpm smoke`) intact. This is what makes `git bisect` meaningful.
- Speculative or exploratory work goes on a branch: `git checkout -b experiment/<name>` (e.g. `experiment/chrome-extension`). Commit messily; if it works merge to `main`, if not delete the branch.
- Feature work: `feat/<name>`. Bug fixes: `fix/<name>`.

## Milestone tags

Tag versions at meaningful milestones so the history has anchor points:

- `v0.0.1-poc` — Phase 0 (end-to-end feasibility) verified.
- `v0.1.0` — Phase 1 (Vite plugin + chat UI + persistent service) shipped.

## Test strategy

- Unit tests: **Vitest**, per-package, in `packages/*/tests/` sibling to `src/`. Run with `pnpm --filter @hover-dev/core test` or `pnpm test` at the root (which fans out across the workspace). Keep `src/` source-only; do not place `*.test.ts` inside `src/`. Current coverage: `packages/core/tests/agents/` (argv dispatcher, claude descriptor, registry).
- Integration / e2e: **Playwright dogfooding**. Crystallized specs land under `examples/basic-app/__vibe_tests__/` and run with standard `@playwright/test`. The agent must not be involved at CI time — only the Playwright script runs. Bootstrap on a fresh machine: `pnpm --filter basic-app exec playwright install chromium`. Run with `pnpm test:e2e`.
- Smoke-level end-to-end (agent in the loop): `pnpm smoke`. This requires a running debug Chrome and the example frontend; it is not part of CI.

## Validation strategy

Before marking work ready:

1. `pnpm typecheck` — fans out to every package.
2. `pnpm test` — Vitest, fans out across packages with tests.
3. The package-scoped smoke or Playwright run that matches the files changed.

# Common commands

```bash
pnpm install              # workspace install (also runs husky install via the `prepare` script)
pnpm typecheck            # tsc --noEmit, per-package
pnpm test                 # vitest, per-package (where present)
pnpm test:e2e             # Playwright dogfood suite — first run needs `playwright install chromium`
pnpm dev:example:basic-app         # http://localhost:5173 — login / counter / todos
pnpm dev:example:e-commerce        # http://localhost:5174 — Amazon-style storefront
pnpm dev:example:event-form        # http://localhost:5175 — eleven rich controls
pnpm dev:example:canvas-paint      # http://localhost:5176 — canvas + DOM toolbar
pnpm dev:example:payment-provider  # http://localhost:5177 — mock third-party popup, no widget
pnpm dev:example:astro-app         # http://localhost:5178 — Astro 5 smoke (counter + todos)
pnpm dev:example:nuxt-app          # http://localhost:5179 — Nuxt 4 smoke (counter + todos)
pnpm dev:example:next-app          # http://localhost:5182 — Next 16 App Router smoke (counter + todos)
pnpm dev:example:webpack-app       # http://localhost:5180 — vanilla webpack 5 smoke
pnpm dev:example:rn-web-app        # http://localhost:5181 — React Native Web smoke
pnpm smoke:chrome         # launch debug-mode Chrome (--remote-debugging-port=9222)
pnpm smoke                # end-to-end: detect agents → CDP preflight → invoke claude
pnpm detect               # list installed coding agents
pnpm verify-widget        # validate that the injected widget reports `data-vibe-test`
pnpm ws-smoke             # exercise the @hover-dev/core WebSocket bridge in isolation
pnpm bench-ttfb [n=5]     # time the LLM-driven loop's first tool_use latency (needs Chrome on :9222 + a dev server). A/B perf changes by running on each branch.
```

```bash
pnpm --filter @hover-dev/core test
pnpm --filter @hover-dev/core typecheck
pnpm --filter vite-plugin-hover typecheck
pnpm --filter basic-app dev
```

# FAQ

## Why is the default model `sonnet` and not `opus`?

A typical browser-driving session with `opus` costs ~5× the equivalent `sonnet` session. Hover is meant to run continuously during dev; default to `sonnet`. Set `HOVER_MODEL=opus` per-invocation when you need it.

## Why does Hover use an isolated debug Chrome instead of attaching to the user's normal browser?

Hover launches its own debug Chrome under `<tmpdir>/hover-chrome` (a persistent user-data-dir, reused across runs) and connects to it over CDP. It deliberately does *not* attach to the user's primary Chrome profile: doing so would require the user to relaunch their everyday browser with `--remote-debugging-port` and would expose every tab, cookie, and extension on their main session to whatever the agent does. The trade-off is honest: the user has to log into the app once inside the debug Chrome, but from that point on the profile dir persists session state across Hover commands and dev-server restarts.

The CDP entry point (`connectOverCDP` against an *already-running* debug Chrome) is still load-bearing: the agent never spawns its own Chromium, never operates a fresh headless context, and always lands on the existing tab whose URL matches the dev-server origin.

## Why is filesystem access disallowed on the agent?

The agent only needs the Playwright MCP server — that is enough to drive the browser end-to-end. Allowing `Bash`, `Edit`, `Write`, `Read`, etc. dramatically widens the blast radius if the prompt is hijacked or if the agent hallucinates a destructive action. The single write path (eventually under `__vibe_tests__/`) is granted by the Node service, not by the agent's tool list.

## Why is the UI in a Shadow DOM?

Two reasons: (1) style isolation from the host app, so Hover's CSS does not bleed into the page under test, and (2) Playwright tests must be able to skip Hover's own DOM — the `data-vibe-test="true"` marker on the Shadow root makes the filter trivial. Tailwind's content scanner does not see inside Shadow DOM; use inline styles or CSS-in-JS.

## Why `--max-budget-usd 0.50`?

A safety belt against runaway prompts. Phase 0 sessions empirically complete a 5-step task on the example frontend for well under $0.10; $0.50 is generous but still catches a runaway loop before it becomes expensive. Tune up only with explicit reason.

## "ERR_REQUIRE_ESM" when loading `@hover-dev/security` under Next?

Symptom: `require() of ES Module .../get-port/index.js from .../mockttp/dist/server/mockttp-server.js not supported`. Chain is `@hover-dev/security` → `mockttp@4.4.2` → `require('get-port')` → `get-port@7.x` (ESM-only). `mockttp` upstream is aware — see [httptoolkit/mockttp#200](https://github.com/httptoolkit/mockttp/issues/200) (open as of 2026-05) — but ships no fix yet.

Workarounds, by preference:

1. **Upgrade to Node ≥ 22.12** — Node added sync `require(ESM)` in 22.12, so the load succeeds out of the box. `@hover-dev/security` declares `engines.node >= 22.12.0` for this reason. Older Node still emits the runtime error.
2. **Pin `get-port` to v6 in your project's overrides**:
   ```json
   { "pnpm": { "overrides": { "get-port": "^6.1.2" } } }
   ```
   (npm: `"overrides"` at top level; yarn: `"resolutions"`.) get-port@6.x is CJS and `mockttp`'s `require()` works.
3. **Remove the `@hover-dev/security` plugin from your `register()` call** if you don't need MITM mode — Hover works fine without it.

We can't fix this from inside `@hover-dev/security`: npm overrides only flow from the consumer's root package.json, so a published dep can't override a sibling dep's resolution.

## "Cannot get schema for 'PrivateKeyInfo' target" when enabling security mode?

Symptom: the widget reports `[hover/mitm] CA generation failed: @peculiar/asn1-schema schema-registry collision`. Root cause: `@peculiar/asn1-schema` keeps its ASN.1 schema definitions on a per-module-instance singleton. When two copies of the package end up in the consumer's `node_modules` (pnpm hoisting + Next 15's module-resolution combine produces this readily), PKI deps register schemas into copy A's registry but the runtime lookup walks copy B's empty one → schema not found.

`@hover-dev/security` declares `@peculiar/asn1-schema@2.6.0` as a direct dependency to give pnpm a strong hint, but inside the consumer's tree mockttp's own sub-deps may still pull in a sibling copy.

Fix from the consumer's root package.json:

```json
{ "pnpm": { "overrides": { "@peculiar/asn1-schema": "2.6.0" } } }
```

(npm: `"overrides"` at top level; yarn: `"resolutions"`.) Then `rm -rf node_modules && pnpm install` to collapse to one copy. Verify with `pnpm why @peculiar/asn1-schema` — should show exactly one resolved version.

The startProxy() loop now detects this exact error message and rewrites it into the fix recipe, so the widget panel shows a useful error instead of a generic `no free port` swallowed-error trail.

Tracking upstream: [PeculiarVentures/asn1-schema#111](https://github.com/PeculiarVentures/asn1-schema/issues/111).

---
> Source: [Hyperyond/Hover](https://github.com/Hyperyond/Hover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
