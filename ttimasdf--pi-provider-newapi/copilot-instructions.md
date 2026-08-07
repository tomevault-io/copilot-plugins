## pi-provider-newapi

> `pi-provider-newapi` is a [pi](https://github.com/earendil-works/pi) coding-agent extension that exposes self-hosted [NewAPI](https://github.com/QuantumNous/new-api) gateways as pi model providers. It targets **pi SDK v0.84.0+** (`@earendil-works/pi-ai` and `@earendil-works/pi-coding-agent`).

# AGENTS.md

`pi-provider-newapi` is a [pi](https://github.com/earendil-works/pi) coding-agent extension that exposes self-hosted [NewAPI](https://github.com/QuantumNous/new-api) gateways as pi model providers. It targets **pi SDK v0.84.0+** (`@earendil-works/pi-ai` and `@earendil-works/pi-coding-agent`).

`index.ts` is the single Pi entry point and exports only the extension factory. Implementation modules live under `src/`; focused tests under `test/` import those modules directly. There is no build step — pi loads the `.ts` entry directly (`package.json` → `pi.extensions`).

## Commands

```bash
npm run typecheck   # tsc --noEmit (checks index.ts, src/, and test/)
npm test            # node --test (discovers test/*.test.ts via Node's TS strip-only loader)
```

Interactive smoke test against the source checkout (needs a TTY; will hang in non-interactive shells):

```bash
../pi-src/pi-test.sh -e ./index.ts
```

Node's strip-only TS loader powers `npm test`: **no TS-only runtime syntax** (no `enum`, no parameter properties, no decorators). Keep types erasable.

## Hard constraints (v0.84.0 contract)

- **Pi owns credentials.** Never read/write `auth.json`, never touch the removed `modelRegistry.authStorage`. Users enter keys via `/login <name>` and remove them via `/logout <name>`.
- **Providers register once with `models: []` + `refreshModels(context)`.** The empty catalog is intentional — it makes the provider selectable in `/login` before any model is discovered, which bootstraps credential entry. Do not re-`registerProvider` per discovery.
- **Discovery is Pi-driven.** `/model` background refresh and `pi update --models` call `refreshModels(context)`. Honor `context.allowNetwork`, `context.force`, `context.signal`, `context.credential`, restore from `context.stored`, and persist through generation-checked `context.publish()`.
- Never copy the API key into provider config, logs, notifications, model definitions, or the model store.

## Implementation map (`src/`)

Data flow: **config + Pi credential → discover → enrich → route API → build model configs → register/persist.**

- **`config.ts`** (`readConfig`/`writeConfigAtomic`/`updateConfig`): stores configuration at `<agentDir>/extension-settings/provider-newapi.json` as `{ version, providers: { <name>: { baseUrl, modelApiOverrides } }, settings }`. `updateConfig` is a serialized read-modify-write (module-level promise queue) with atomic temp-file+rename, so concurrent provider setup operations never clobber each other's entries. Malformed config is backed up to `.bak` and reset.
- **`migration.ts`**: owns legacy configuration compatibility. `getConfigVersion()` unifies the active path and on-disk schema version; migration moves `<agentDir>/extensions/provider-newapi.json`, upgrades pre-versioned schema `0` files to schema `1`, and warns about removed extension-owned model overrides.
- **`http.ts`** (`fetchWithTimeout`): combines a local timeout with `context.signal` via `AbortSignal.any`; throws a `NewAPIError` tagged `aborted | timeout | auth | http | payload | network`.
- **`models.ts`** (pure, exported, tested): parses `/v1/models` and ratio configuration, enriches models from Pi's built-in catalog, applies regex `modelApiOverrides`, computes costs, and builds provider model configs. Metadata and compatibility overrides belong to Pi's `models.json`.
- **`generated-models.ts`**: builds unknown-model `modelOverrides` templates and atomically writes `<agentDir>/models-generated.json`; it never edits Pi's user-owned `models.json`.
- **`discovery.ts`**: fetches ratio config (best-effort) + `/v1/models` (required), reads the API key from `context.credential`, calls `buildProviderModels`, and implements `context.stored` cache fallback plus generation-checked persistence through `context.publish()`.
- **`provider.ts`**: registers configured NewAPI providers with `models: []` and dynamic `refreshModels` callbacks.
- **`commands.ts`**: registers the add/remove/list commands.
- **`extension.ts`**: composition root for startup provider registration, onboarding, and commands.
- **`index.ts`**: stable Pi entry point that exports only the extension factory. Keep `package.json` → `pi.extensions` pointed at `./index.ts` so internal `src/` paths do not change the startup display label.

## Commands (user-facing)

- `/newapi-provider-add [name]` — prompt name + base URL, persist config, register live, tell the user to run `/login`. Only a best-effort unauthenticated reachability check; auth verification happens later in `refreshModels`.
- `/newapi-provider-remove [name]` — unregister + delete config entry. Warns to run `/logout` first because Pi exposes no extension-safe credential deletion. Never edits `auth.json`.
- `/newapi-provider-list` — uses `ctx.modelRegistry.getProviderAuthStatus(name).configured`; never prints secrets.
- `/newapi-generate-models-json` — reloads the currently available catalogs and writes unknown-model metadata templates to `<agentDir>/models-generated.json`; users manually merge relevant entries into Pi's `models.json`. On Pi versions where registry refresh does not trigger provider discovery, open `/model` first.

## Conventions

- Tabs for indentation; keep the file dependency-free beyond the two pi peer packages + Node built-ins.
- Extract pure, deterministic logic as exported functions and unit-test it; keep I/O (fetch, fs, Pi APIs) thin.
- Keep `package.json` `version` and the top `CHANGELOG.md` entry in sync; update both READMEs (`README.md` + `README_cn.md`) when user-facing behavior changes.

---
> Source: [ttimasdf/pi-provider-newapi](https://github.com/ttimasdf/pi-provider-newapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
