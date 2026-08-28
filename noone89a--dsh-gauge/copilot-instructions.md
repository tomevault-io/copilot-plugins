## dsh-gauge

> A DeepSeek Harness (DSH) plugin that fixes the cache-hit rate display in the Web UI

# dsh-gauge

A DeepSeek Harness (DSH) plugin that fixes the cache-hit rate display in the Web UI
and adds a session usage panel.

## Repository rules

- Keep the package self-contained: every runtime import resolves from `node_modules`
  or `src/`. No absolute workstation paths, no `file:`/`link:` dependency specs.
- The host side (`src/index.ts`) is a function plugin: named exports `name`,
  `inject`, `Config`, `apply`. Never add a default export.
- The browser side lives behind the `./client` export; components receive data only
  through the four props shares (runtime seats / owner / store / inject). Business
  components never touch `ctx`.
- The client entry is served verbatim by dsh-client-modules, so `npm run build`
  MUST end with the `scripts/build-client.mjs` bundling step that emits the
  `window.__ModuleLoader__.load({ id, factory })` format. A raw tsc ESM output
  breaks the whole page at boot. Never publish a `lib/client.js` that contains
  top-level `import`/`export`.
- Before changing slot registrations, read the official slot system standard:
  `packages/client/AGENTS.md` in the deepseek-harness repo.
- Run `pnpm run typecheck`, `pnpm test`, and `pnpm run build` before committing.

---
> Source: [noone89A/dsh-gauge](https://github.com/noone89A/dsh-gauge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
