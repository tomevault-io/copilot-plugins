## dsh-llm-oauth

> Standalone DeepSeek Harness plugin (`dsh-llm-oauth`). It is **not** part of the `deepseek-ai/deepseek-harness` monorepo.

# AGENTS.md

Standalone DeepSeek Harness plugin (`dsh-llm-oauth`). It is **not** part of the `deepseek-ai/deepseek-harness` monorepo.

Official plugin contract: [first plugin](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/index.md), [publish](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/publish.md).

## Contract

- Function plugin: export `name`, `inject`, `Config`, `apply`. **No `export default`.**
- Bundle: `package.json` `dsh.bundle.patch` → `cordis.patch.yml`.
- Optional web client: `package.json` `dsh.client` → `lib/client.js` (`settings.section`, id `oauth`, order 12).
- Git installs must build with `scripts/prepare.mjs` (tsdown host + client; do not typecheck DSH peers). Client needs `lightningcss`; skip it quietly if missing.
- `@deepseek-ai/*` are peers from the host profile. `@earendil-works/pi-ai` is this package's dependency (current `^0.84.2`).
- Registrations go through `ctx.llm.registerAdapter` / `registerConfigurableProviders` / `ctx.commands.register` (when present).
- Settings namespace: `llm-oauth` (`catalog` + enabled `providers` dict). Default is dormant (`providers: {}`).
- Optional HTTP API under `/dsh-llm-oauth/*` when `webServer` is present (`ctx.inject(['webServer'], …)`).
- Do not also register the same provider id under `llm-pi-ai` (`DUPLICATE_ADAPTER`). Subscriptions here; API keys stay on first-party plugins.

## Catalog / models

- Model table is **only** `@earendil-works/pi-ai`. Do not invent local model ids.
- Do not hardcode extra model ids; bump `@earendil-works/pi-ai` when the catalog needs to move.
- Web `openai-codex` must auto-pick `device_code` (no localhost `:1455` callback). ChatGPT still requires “Enable device code authorization for Codex” on their side.

## `/oauth`

Actions: `status` | `list` | `enable` | `disable` | `login` | `logout`.

`login` **must return as soon as** a URL or user code is available. DSH only renders when the handler returns; do not await the device-code poll. Poll in the background; `/oauth status` / Settings refresh report the watch.

## Commands

```sh
pnpm install
pnpm test
pnpm run build          # host (tsdown.config.ts) + client (tsdown.client.config.ts)
node bin/login.mjs --list
```

---
> Source: [ziyou979/dsh-llm-oauth](https://github.com/ziyou979/dsh-llm-oauth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
