## dsh-aemeath-pet

> Out-of-tree DeepSeek Harness plugin (host + Web client bundle): a desktop pet for the DSH web GUI. The harness checkout is expected at `../../deepseek-harness` (adjust the `link:` devDependencies in package.json if it lives elsewhere).

# AGENTS.md

Out-of-tree DeepSeek Harness plugin (host + Web client bundle): a desktop pet for the DSH web GUI. The harness checkout is expected at `../../deepseek-harness` (adjust the `link:` devDependencies in package.json if it lives elsewhere).

## Layout

```
src/index.ts         host entry: name/inject/apply — serves the spritesheet at /aemeath-pet/spritesheet.webp
src/client/          browser half (single-file bundle, no React)
  index.ts           apply: sessions wiring → pet state, locale, burst/bubble dispatch
  pet.ts             AemeathPet engine: canvas animation, loop-driven random stunts, drag/menu/prefs
  state.ts           pure facts→mood/burst derivation (unit-tested)
  atlas.ts           Codex Pet V2 grid layout (rows/frames/fps per animation)
  locales.ts         zh/en dictionaries + LocaleNamespaceMap augmentation
  styles.ts          injected CSS (ap-* prefixed)
assets/              spritesheet-v2-r2.webp (from aemeath-codex-pet R2, Apache-2.0)
lib/                 built artifacts, COMMITTED (tarball installs run without a build)
tests/               state.spec (pure) + pet.spec (jsdom engine specs)
```

## Contracts with the harness (do not drift)

- The client bundle registers via `window.__ModuleLoader__.load({ id: 'dsh-aemeath-pet', factory })` and exports `{ inject, apply }`; `dsh.client` (platform web, inject sessions+locale) is declared in package.json, served at `/plugins/dsh-aemeath-pet/client.js`.
- The host half injects `webServer` and registers one exact route; the client fetches the asset from it. Nothing else crosses the wire.
- The pet consumes the current session's `ConversationSnapshot` through `ctx.sessions` (list + binding). Stunt scheduling lives in the rAF loop (`nextStuntAt`) — never move it back to setTimeout chains; the loop is the proven-alive heartbeat.
- The @-token-free DOM overlay must stay pointer-transparent (`pointer-events: none` on the root, auto only on canvas/menu/paw) so the pet never blocks the app.

## Check ladder

`pnpm run check` (typecheck + vitest + build) must be green before every commit; rebuild keeps `lib/` in sync with `src/`. Tests: statements/branches/lines 100% per source file; engine specs drive the rAF loop manually (tick()).

## Copy

Product copy is Chinese (locales.ts is the key-set source of truth); code comments, JSDoc, and the English README are English.

---
> Source: [culture-flask/dsh-aemeath-pet](https://github.com/culture-flask/dsh-aemeath-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
