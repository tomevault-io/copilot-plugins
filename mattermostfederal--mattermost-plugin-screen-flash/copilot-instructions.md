## mattermost-plugin-screen-flash

> Mattermost plugin that flashes a full-viewport overlay on incoming desktop notifications (DMs and/or @mentions). Designed for high-noise environments (SOC operations, trading floors) where audio alerts get missed. Configurable color, count, duration, and focus-awareness. See [README.md](README.md) for the user-facing description.

# Screen Flash Alert — Project Notes

## What this plugin does

Mattermost plugin that flashes a full-viewport overlay on incoming desktop notifications (DMs and/or @mentions). Designed for high-noise environments (SOC operations, trading floors) where audio alerts get missed. Configurable color, count, duration, and focus-awareness. See [README.md](README.md) for the user-facing description.

## Where the interesting code lives

- [webapp/src/flash.ts](webapp/src/flash.ts) — the overlay engine. Single full-viewport `<div>` with `pointer-events:none`, opacity animated via `setTimeout`. `flashScreen()` is **debounced** — concurrent calls during an in-flight flash are dropped on purpose (a burst of messages must not stack into unbounded visual noise). `cancelFlash()` clears timeouts and zeroes opacity.
- [webapp/src/index.tsx](webapp/src/index.tsx) — registers the `registerDesktopNotificationHook`, the `config_update` WebSocket listener, and the `test_flash` listener. Always returns `{args}` unchanged from the hook — this plugin only **adds** visual feedback; it never suppresses or modifies notifications.
- [server/plugin.go](server/plugin.go) — broadcasts the live plugin configuration to all connected clients via `PublishWebSocketEvent("config_update", ...)`. Done because the webapp cannot read plugin config from REST on Mattermost Cloud, so the server pushes it on (a) `OnConfigurationChange` (broadcast to all) and (b) `UserHasLoggedIn` (broadcast to that user). Webapp keeps its own copy and updates immediately on receipt — admins do not need users to refresh.
- [server/command.go](server/command.go) — `/screenflash test|status`. `test` publishes a `test_flash` WebSocket event scoped to the requesting user only.

## Conventions that aren't obvious from the code

- **Numeric config is clamped both server- and client-side.** `FlashCount` 1–10, `FlashDuration` 50–500 ms. Either side may receive out-of-range values (System Console doesn't enforce), so both sides clamp defensively. Don't remove either set of clamps.
- **The overlay's z-index is the max signed 32-bit int** (`2147483647`) so it sits above all Mattermost UI layers. Don't lower it.
- **`pointer-events:none` is load-bearing.** Operators must keep clicking through to the underlying UI while the flash runs.

## Testing setup (non-obvious bits)

- **Backend** uses `plugintest.API` mocks (testify-style). Mock setup needs an explicit `defer api.AssertExpectations(t)` per test. The `OnConfigurationChange` test sets zero values in `LoadPluginConfiguration` to exercise the default-fallback branch — that's the only realistic way to hit those paths.
- **Frontend** uses Playwright in two modes:
  - `*.spec.ts` runs in Node (Playwright's plain test runner).
  - `*.pw.tsx` runs as Component Tests in real Chromium via `@playwright/experimental-ct-react`, with a Vite build pipeline. The CT mount happens inside an iframe-like setup but `document.body` is the page body (overlay creation works as it would in prod).
- **Coverage plumbing** is hand-rolled because `vite-plugin-istanbul@6+` is ESM-only and Playwright's config loader is CJS:
  - [webapp/vite-istanbul-plugin.ts](webapp/vite-istanbul-plugin.ts) is a 50-line replacement using `babel-plugin-istanbul` (CJS-friendly). Only runs when `COVERAGE=1`.
  - [webapp/coverage-fixture.ts](webapp/coverage-fixture.ts) extends the CT `page` fixture to capture `window.__coverage__` and write `.nyc_output/ct-*.json` per test.
  - Spec tests are wrapped with `c8` writing istanbul-format JSON to the same `.nyc_output`. `nyc report` merges both.
  - **Gotcha**: Vite caches the CT bundle in `playwright/.cache/`. Switching between coverage and non-coverage runs reads stale instrumentation. The `test:pw-ct-coverage` npm script wipes the cache first — keep that.
- Adding a new `*.pw.tsx` test: import `test` and `expect` from `'../coverage-fixture'` (not from `@playwright/experimental-ct-react`) so the coverage capture fires.

## Build / dev workflow

- `make dist` — build the `.tar.gz` bundle (also runs `apply` → regenerates `server/manifest.go` and `webapp/src/manifest.ts` from `plugin.json`, both gitignored).
- `make check-style` — go vet + golangci-lint + eslint + tsc.
- `make test` — backend Go tests + frontend Playwright (both runners).
- `make coverage` — backend (gotestsum coverage) + frontend (c8 + CT istanbul + nyc merge).
- `make sbom-audit` — CycloneDX SBOMs + Grype scan. Suppressions live in [.grype.yaml](.grype.yaml).
- `make codeql-analyze` + `make security-gate` — CodeQL SAST + SARIF gate. Downloads ~700 MB CodeQL bundle to `build/codeql/` on first run.
- `make docker-setup` then `make deploy` — local Mattermost via OrbStack/Docker.

## Things to leave alone unless asked

- The Makefile is template-derived. The user generally prefers it minimally modified. Past edits: `make clean` was extended to also remove `webapp/.nyc_output` and `webapp/playwright/.cache`.
- The `.grype.yaml` `ignore` list is currently empty — all known high-severity findings have been patched (not suppressed). Prefer patching over adding suppressions; if you must suppress, include a `reason` string.

---
> Source: [MattermostFederal/mattermost-plugin-screen-flash](https://github.com/MattermostFederal/mattermost-plugin-screen-flash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
