## pi-droid-sdk

> pi provider extension that registers Factory Droid models under the `factory` provider via `@factory/droid-sdk`. Agent work succeeds when changes preserve pi-native model/thinking/session behavior, keep Factory API keys out of repo state and logs, and pass local validation.

# AGENTS.md

## Purpose

pi provider extension that registers Factory Droid models under the `factory` provider via `@factory/droid-sdk`. Agent work succeeds when changes preserve pi-native model/thinking/session behavior, keep Factory API keys out of repo state and logs, and pass local validation.

## Repository map

- `src/index.ts` — registers the pi extension, provider, fallback warnings, bridge hooks, question tool, and session cwd.
- `src/model-discovery.ts` — discovers Factory models via ephemeral Droid session init, builds pi model metadata, fallback catalog.
- `src/droid-provider.ts` — streams through local `@factory/droid-sdk` sessions, bridges pi tools, maps stream events to pi assistant events.
- `src/droid-pi-tool-bridge.ts` — exposes active pi tools to Droid through `createSdkMcpServer`.
- `src/droid-question-tool.ts` — bridge-exposed `droid_ask_question` pi UI tool + `askUserHandler` wiring.
- `src/droid-permissions.ts` — `permissionHandler` wiring for Droid tool confirmations.
- `src/droid-session-cwd.ts` — session cwd tracking for Droid runs.
- `src/context.ts` — pi message → Droid prompt conversion.
- `test/**/*.test.ts` — Vitest coverage.

## Validation

```bash
npm run typecheck
npm test
```

## Auth

Factory auth uses pi-native API-key resolution for provider `factory`: CLI `--api-key`, stored `~/.pi/agent/auth.json` from `/login`, then `FACTORY_API_KEY`. Requires `droid` on PATH (SDK spawns it).

## Env

- `PI_DROID_PI_TOOL_BRIDGE` — enable/disable pi tool bridge (default: enabled)
- `PI_DROID_EXPOSE_BUILTIN_TOOLS` — expose overlapping builtins through bridge
- `PI_DROID_AUTONOMY_LEVEL` — `off|low|medium|high` for Droid native tool autonomy (default: `high`)

---
> Source: [bentossell/pi-droid-sdk](https://github.com/bentossell/pi-droid-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
