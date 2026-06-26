## booster-plugins

> validates the rest (`capabilities.md` in `../booster-framework/docs/`).

# booster-plugins

Monorepo of the **internal** plugins that ship with `steambooster`:
`booster-checkout` (popup with topup / payment / support / orders — the
main UI in Steam's header) and `booster-addfunds` (page-mod for
`store.steampowered.com/steamaccount/addfunds`). Each package builds
to a standalone IIFE bundle and is delivered to the native injector
through the production v2 manifest's `requiredPlugins[]` block, signed
with the project Ed25519 key.

> **Global project conventions are owned by the operator and not
> mirrored here.** This file covers plugin-local work only. The
> plugin-API contract lives at `../booster-framework/docs/`.

## Per-package layout

```
booster-plugins/
├── packages/
│   ├── booster-checkout/        # Main-context popup plugin
│   │   ├── src/
│   │   │   ├── index.ts          # register() entry
│   │   │   ├── urls.ts           # plugin-owned URL constants (allowed here)
│   │   │   ├── i18n.ts
│   │   │   ├── generated/        # typesafe-i18n codegen
│   │   │   ├── main/             # main-context logic
│   │   │   ├── lib/              # shared helpers
│   │   │   └── shared/           # types used by both src/ and popup-svelte/
│   │   ├── popup-svelte/         # Svelte 5 popup UI
│   │   │   ├── App.svelte
│   │   │   ├── main.ts
│   │   │   ├── components/
│   │   │   ├── styles/
│   │   │   ├── lib/
│   │   │   └── __tests__/
│   │   ├── scripts/build-popup.ts  # Svelte → inlined HTML string
│   │   ├── strings/ru.json         # checkout + general namespaces
│   │   ├── assets/
│   │   ├── tests/
│   │   ├── build.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── booster-addfunds/        # Web-context page-mod plugin
│       ├── src/               # urls.ts, i18n.ts, generated/, …
│       ├── strings/ru.json    # addfunds + general namespaces
│       ├── assets/
│       ├── tests/
│       ├── build.ts
│       ├── package.json
│       └── tsconfig.json
│
├── docs/
│   ├── contributing.md     # code style, test discipline, PR process
│   └── release-process.md  # tagging, release flow, injector pickup
├── scripts/                # shared helpers
├── tools/                  # shared build helpers
├── package.json            # bun workspaces root
├── tsconfig.json
├── CLAUDE.md               # (this file)
└── README.md               # external-facing intro
```

## Framework dependency

This repo depends on the public `booster-framework` repo
(`STEAMBALANCE`). It must be cloned next to this one so the `file:`
dependency resolves:

```
<workspace>/
├── booster-framework/         ← public runtime + window.sb API (dependency)
└── booster-plugins/           ← THIS repo
```

Each package declares
`"@steambalance/booster-framework": "file:../../../booster-framework"` in its
`package.json`; the dependency resolves against the neighbouring clone
on disk rather than through the workspace, because the framework
lives in a separate repo. That clone must exist at
`../booster-framework` before `bun install` runs here.

**Standalone work is OK** for plugin-local changes: tweak popup CSS,
add a BC message, fix a string. `bun test` (per-package) covers the
unit-level behaviour. End-to-end Steam QA runs under the native
injector (operated separately), which rebuilds the plugin bundle,
picks up the new sha256, and hot-updates the running plugins inside
Steam.

## Plugin conventions

**Commit messages.** Conventional Commits / git-flow style (global rule
for all 4 repos; canonical in `../booster-injector/CLAUDE.md`). Use a type
prefix (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, `build:`,
`ci:`, `perf:`, …) with an optional scope, e.g. `fix(checkout): …`. Subject
in lowercase — don't capitalize the first word; capitals only where the
language demands them (proper nouns, acronyms). Keep it laconic and clear,
not a wall of text; English only. A body is optional and brief — add one
only when the *why* isn't obvious from the subject.

**Plugin id rules** (enforced by the native injector's manifest
loader):

- `booster-` prefix → reserved for the internal plugins in this repo
  (`booster-checkout`, `booster-addfunds`). Goes into `requiredPlugins[]`.
- Non-`booster-` prefix → external / vetted plugins. Goes into
  `approvedPlugins[]` of the manifest. Authors use the
  `../booster-plugin-template/` starter.
- The `Auth` capability is **gated to `booster-`-prefix plugins** — manifest
  loader rejects an `approvedPlugins[]` entry that requests `auth`.

**Strings.** Per-package, under each plugin's `strings/ru.json`:

- `packages/booster-checkout/strings/ru.json` — top-level keys: `checkout`
  + `general`.
- `packages/booster-addfunds/strings/ru.json` — top-level keys: `addfunds`
  + `general`.

Call via `LL.checkout.<key>()` or `LL.addfunds.<key>()`. To add a
string: edit the relevant `strings/ru.json`, run the per-package
`gen-strings`, call the generated accessor.

The `no-hardcoded-ru` guard runs per-package — Cyrillic
literals outside `*/generated/*` are forbidden without an
`// strings-allow-cyrillic` pragma.

**URLs.** This repo is the **only** place where URLs may be hardcoded
in source — specifically in `packages/<plugin>/src/urls.ts`. The
`no-hardcoded-urls` guard's allowlist matches exactly that pattern;
URLs anywhere else (popup-svelte, lib/, main/) are rejected. Use the
constants exported from `urls.ts`.

**Capability declaration.** Each plugin's `register()` call passes a
`PluginManifest` listing required capabilities. The manifest loader
on the C++ side validates `auth` gating; the framework on the TS side
validates the rest (`capabilities.md` in `../booster-framework/docs/`).
Request the **minimum** set — `booster-checkout` needs `Ui, Steam, Configs,
Bus, Auth` (full-featured popup); `booster-addfunds` needs `Ui, Steam,
Configs, Bus, Pages` (page-mod that registers via `sb.pages`, no Auth).
The full capability set is seven: `ui, steam, configs, auth, bus, pages,
keys` — request only what the plugin actually calls.

## Build / test (per-package)

```pwsh
cd booster-plugins
bun install                                 # installs all workspaces
bun run build                               # --filter '*' across packages
bun run test                                # per-package suites (each bunfig)

# Per-package:
bun --cwd packages/booster-checkout run build
bun --cwd packages/booster-checkout test
bun --cwd packages/booster-addfunds run build
bun --cwd packages/booster-addfunds test
```

> A bare `bun test` from the monorepo root does **not** work: `bunfig.toml`
> (which registers the `bun-plugin-svelte` + `happy-dom` preload for the
> popup-svelte runes tests) is per-package, so a root run skips it and the
> Svelte tests fail with `$state is not defined`. Always run per-package
> (`bun run test` dispatches to both; CI does the same).

`booster-checkout` popup-build chain: `scripts/build-popup.ts` runs
`bun-plugin-svelte` + `svelte-preprocess` (PostCSS / cssnano in
production) and composes an all-in-one HTML string (inlined SVG, IIFE
JS, minified via bun's built-in `minify` in production). The result
lands in the plugin bundle as `__SB_POPUP_HTML__` via a build-time
`define` substitution. No separate `html-minifier-terser` or JS
obfuscator is used.

## Test status

All suites are green. `popup-svelte/__tests__/` render via the
`tests/setup-svelte-plugin.ts` preload (redirects Svelte's server entry
to the client runtime so `mount()` / runes work) backed by `happy-dom` —
there is no SSR-failure baseline. `bun --cwd packages/booster-checkout test`
runs clean (popup-svelte 176/0, package total 317/0); `booster-addfunds`
is 28/0.

## See also

- `../booster-framework/docs/plugin-contract.md` — `register()`,
  `PluginManifest`, capability matrix.
- `../booster-framework/docs/capabilities.md` — full capability reference,
  `Auth` sensitive-flag.
- `../booster-framework/CLAUDE.md` — framework-local conventions (when
  touching the API you depend on).
- `docs/contributing.md` — code style, PR process.
- `docs/release-process.md` — how the native injector picks up new
  plugin versions.
- `README.md` — external-facing intro.

---
> Source: [STEAMBALANCE/booster-plugins](https://github.com/STEAMBALANCE/booster-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
