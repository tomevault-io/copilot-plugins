## spritely

> Guidance for AI coding agents working in this repository. The reader is assumed

# AGENTS.md

Guidance for AI coding agents working in this repository. The reader is assumed
to know nothing about the project.

## Project overview

`@wangjunjian/dsh-spritely` (**Spritely**) is a **DeepSeek Harness
(DSH) client plugin** that mounts a floating work-state mascot into the DSH
`web` UI. The sprite animates with the agent's live activity (idle, thinking,
writing, working, waiting, error, done), tracks the cursor with its gaze, can
be dragged anywhere (position persisted), offers switchable characters
(Blob/Bot/Cat/Ghost), and can recolor the app background (solid, gradient,
image URL, or local upload). The plugin also contributes a **selection
toolbar**: selecting text inside a conversation message floats an action bar
(copy, read aloud via SpeechSynthesis, quote-and-ask into the composer draft).

It is a **surface** plugin: the host entry (`src/index.ts`) exports an empty
`apply()` — there is no host-side behavior, no RPC channel, and no Config
schema. The package still ships a `cordis.patch.yml` declared via
`dsh.bundle.patch` so that `dsh plugin add` auto-inserts its loader row
(`id: ui-sprite`); the web half is picked up from the `dsh.client` metadata.

## Technology stack

- **Language**: TypeScript (strict), ESM (`"type": "module"`), React 18.
- **Runtime**: browser half only (`src/client/`), built on the DSH client slot
  system (`ctx.slots.inject` / `ctx.slots.register`) and the locale service.
- **Package manager**: pnpm 10 (pinned via `packageManager`; lockfile committed).
- **Build**: `tsc` (ES2024, bundler resolution) + `tsdown` (Rolldown-based
  bundler) + `lightningcss` for CSS Modules.
- **Lint/format**: Biome 2 (`biome.json`).
- **Tests**: Vitest 4 + jsdom + `@testing-library/react`.
- **Peer dependencies**: the `@deepseek-ai/dsh-*` packages are regular registry
  packages (release candidates); no local checkout of `deepseek-harness` is
  needed for development.

## Repository layout

```
src/
  index.ts                      # Host loader entry: empty apply() (no host behavior)
  invariant.ts                  # Cordis invariant companion (no-op installer, reserves ownership)
  css-modules.d.ts              # Ambient declarations for *.module.css imports
  client/
    index.ts                    # Client entry: locale dicts + shell.overlay registration
    sprite-state.ts             # Work-state source: derives SpriteActivity from the sessions service
    background-source.ts        # Persisted background source + BackgroundPresenter (CSS var applier)
    backgrounds.ts              # Background value shapes and presets
    sprite-kind-source.ts       # Persisted selected-character source (localStorage)
    sprite-position-source.ts   # Persisted dragged anchor position (localStorage)
    selection-toolbar.ts        # Selection tracker (snapshot store) + Markdown quote formatter
    sprites.tsx                 # SpriteKind registry: per-character SVG/pose metadata
    SpriteMascot.tsx            # The mascot component (menu, drag, gaze, poses)
    SpriteMascot.module.css     # CSS Modules next to their component
    SelectionToolbar.tsx        # Floating copy/read-aloud/ask bar over in-conversation selections
    SelectionToolbar.module.css
tests/
  setup.ts                      # window.__ModuleLoader__ polyfill + registry-backed require
  apply.client.spec.tsx         # Entry integration test: real Cordis Context + slot registry
  *.client.spec.ts(x)           # Source and component tests (jsdom via pragma)
images/                         # README screenshots (published to npm)
cordis.patch.yml                # Patch layer inserting the ui-sprite loader row
tsdown.config.ts                # Browser-bundle build (lazy-CJS factory for the web shell)
vitest.config.ts                # Test environments and setup
biome.json                      # Lint/format rules
.github/workflows/ci.yml        # CI: lint, typecheck, test, build
.github/workflows/release.yml   # npm publish on GitHub release (trusted publishing)
lib/                            # Build output (gitignored); do not edit by hand
```

## Build and test commands

```sh
pnpm install           # install dependencies (pnpm 10); also runs `prepare` (a build)
pnpm run typecheck     # tsc --noEmit
pnpm run build         # tsc -b && tsdown  (two-step, see below)
pnpm run test          # vitest run
pnpm run test:watch    # vitest
pnpm run lint          # biome check .
pnpm run lint:fix      # biome check --write .
pnpm run format        # biome format --write .
```

CI (`.github/workflows/ci.yml`) runs `lint`, `typecheck`, `test`, `build` in
that order on Node 22 with `pnpm install --frozen-lockfile`. Keep all four
green before considering a change done.

### Build pipeline (important)

The build is deliberately two-step:

1. `tsc -b` compiles `src/` to JS + declarations under `lib/types/`
   (`outDir: lib/types`). Source imports use `.ts` extensions, rewritten to
   `.js` on emit (`rewriteRelativeImportExtensions`).
2. `tsdown` (see `tsdown.config.ts`) rebundles `lib/types/client/index.js`
   into `lib/client.js`: a single CJS file wrapped in a
   `window.__ModuleLoader__.load({ id, factory })` banner/footer so the DSH web
   shell can load it. It also bundles the node-half entries (`lib/index.js`,
   `lib/invariant.js`) as ESM.
3. Only specifiers in the web shell's frozen module table (`react`,
   `react-dom`, `@deepseek-ai/cordis`, `@deepseek-ai/dsh-client-ui-slots`,
   `@deepseek-ai/dsh-client-runtime/client`, etc. — see `PLATFORM_MODULES`)
   stay external; **everything else is inlined**. A "purity gate" plugin throws
   at build time if the client bundle value-imports any other
   `@deepseek-ai/*` package (type-only imports are erased and therefore fine).
4. CSS Modules (`*.module.css`) are compiled by a custom tsdown plugin using
   lightningcss (`[hash]_[local]` class pattern, minified) and emitted as a
   virtual module that injects a `<style data-plugin-css>` tag at runtime.
   `tsc` does not copy stylesheets, so the plugin maps emitted `lib/` paths
   back to `src/` when loading CSS.

`package.json` declares the plugin via the `dsh` field: `dsh.bundle.patch`
points at `cordis.patch.yml` (a one-row insert registering the plugin id
`ui-sprite` so `dsh plugin add` auto-activates it), and `dsh.client` declares
the `web` platform plus the injected runtime packages.

### Release

`.github/workflows/release.yml` publishes to npm on a published GitHub release
using trusted publishing (OIDC, `--provenance`). The release tag (without the
`v` prefix) must equal `package.json`'s `version`; the workflow fails
otherwise. Bump `version` before cutting a release.

**Release trigger rule: whenever `package.json`'s `"version"` is changed — by
the user or by an agent — run the full release sequence below.** The version is
always read from `package.json`, never typed by hand. Pushing a tag alone does
NOT trigger publishing; a GitHub Release object must be created:

```sh
version="$(node -p "require('./package.json').version")"
git commit -am "release: v${version}"
git tag "v${version}"
git push origin main --tags
# creating the Release is what fires the publish workflow:
gh release create "v${version}" --title "v${version}" --notes "<changes>"
# verify:
gh run watch --workflow=Publish
npm view @wangjunjian/dsh-spritely version
```

Trusted publishing must be registered once on npmjs.com (package Settings →
Trusted Publishers → GitHub Actions: repo `wang-junjian/spritely`, workflow
`release.yml`); without it the publish step fails with `ENEEDAUTH`. The package
lives under the publisher's personal `@wangjunjian` scope.

## Code style guidelines

- **Biome** is the single source of truth: 2-space indent, single quotes,
  trailing commas, 120-column line width (see `biome.json`). Lint preset is
  `recommended`; `noNonNullAssertion` and `noSvgWithoutTitle` are turned off.
- **Imports use `.ts` extensions** for local files, e.g.
  `import { ... } from './locales.ts'` — the tsconfig uses bundler resolution
  with `allowImportingTsExtensions` + `rewriteRelativeImportExtensions` (the
  `.ts` becomes `.js` on emit). Vitest resolves the `.ts` sources directly.
- **Strict TypeScript** (`noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`,
  etc.); avoid `any`. Type-only imports of `@deepseek-ai/*` packages are used
  liberally — they are erased at compile time and keep the bundle pure.
- **JSDoc comments** in English on exported symbols. UI copy is **bilingual**:
  add new strings to both `zh` and `en` dictionaries in `src/client/locales.ts`
  under the `sprite` namespace (the `en` dictionary is keyed by the `SpriteKey`
  type, so a missing translation is a compile error).
- State is held in **snapshot stores** (`createSnapshotStore` from
  `@deepseek-ai/dsh-client-runtime/client`), exposed to React via
  `useSyncExternalStore`; sources with a `persist` option are backed by
  localStorage and must validate rehydrated values (storage is untrusted).
- Slot components receive dependencies through an `inject()` face
  (`SpriteMascotInjected`) rather than importing services directly.
- Make minimal, scoped changes; match the surrounding style.

## Testing instructions

- Run `pnpm run test` (Vitest 4). Test files live in `tests/` and import the
  sources from `../src/...` with `.ts` extensions.
- **Environments**: Node by default; client specs opt into **jsdom** with a
  `// @vitest-environment jsdom` pragma on the first line. (Vitest 4 removed
  `environmentMatchGlobs` — do not re-add it to `vitest.config.ts`.) Name
  client tests `*.client.spec.{ts,tsx}` anyway for consistency.
- `globals: false` — import `describe`/`it`/`expect`/`vi` from `vitest`
  explicitly.
- `tests/setup.ts` installs a `window.__ModuleLoader__` polyfill. Client
  bundles (`@deepseek-ai/*/client`, including this package's own
  `lib/client.js` via the pnpm self-link) self-register into the loader
  registry and **export nothing through CJS**, so named imports from them are
  always `undefined` in tests. To use their real exports (as
  `tests/apply.client.spec.tsx` does), retrieve them through the registry:
  `loader.require('@deepseek-ai/dsh-client-runtime/client')` — the require
  bridge loads the bundle on first access. Where a test only needs a piece of
  the runtime, prefer `vi.mock(...)` with a minimal fake instead (as
  `tests/sprite-mascot.client.spec.tsx` does for `createSnapshotStore`).
- Component tests use `@testing-library/react` with `cleanup()` in `afterEach`.

## Runtime architecture

- **Entry** (`src/client/index.ts`): registers the `sprite` locale namespace,
  then `ctx.slots.inject('shell.overlay', ...)` waits for ui-layout to declare
  the overlay slot and registers two surfaces into it — the mascot (`sprite`)
  and the selection toolbar (`selection-toolbar`). Declared services:
  `inject = ['slots', 'sessions', 'workspaces', 'locale', 'conversation']`.
- **Sources** (created per slot-declaration lifetime, disposed with it):
  - `sprite-state.ts` derives `SpriteState` (`activity` + `toolName`) from the
    standard `sessions` service's conversation snapshot — no cordis events, no
    cross-plugin state.
  - `background-source.ts` persists the app background; `BackgroundPresenter`
    projects it onto the `--dsw-alias-bg-base` CSS variable, recoloring every
    surface layer. Local image uploads are capped (`MAX_UPLOAD_BYTES`) to stay
    under the localStorage quota.
  - `sprite-kind-source.ts` / `sprite-position-source.ts` persist the chosen
    character and the dragged anchor position.
  - `selection-toolbar.ts` tracks the live text selection (debounced
    `selectionchange`; qualifies only non-collapsed selections inside
    `[data-chat-flow]`; hides on outside pointerdown, Escape, and scroll). The
    toolbar's quote-and-ask action resolves the current session's input facade
    (`ctx.sessions.scope(current)` → `conversation.input.for(actx)`), appends
    the selection as a Markdown quote block via `setDraft`, then focuses the
    composer textarea (`[data-composer-card] textarea`) itself — the platform
    deliberately does not steal focus for external drafts.
- **Invariant companion** (`src/invariant.ts`): a Cordis plugin
  (`name: spritely-invariant`, `inject: ['invariants']`) that reserves
  this package's ownership with the invariants service. Its installer is a
  deliberate no-op — the plugin emits no cordis events and owns no cross-plugin
  mutable state, so derivation and interaction behavior are asserted directly
  by this package's specs.

## Security considerations

- All persisted state is localStorage-only and rehydrated through normalizers
  that reject malformed values — keep it that way (storage is untrusted).
- The plugin has no RPC surface and performs no network access; image
  backgrounds load only user-supplied URLs/data URLs.

---
> Source: [wang-junjian/spritely](https://github.com/wang-junjian/spritely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
