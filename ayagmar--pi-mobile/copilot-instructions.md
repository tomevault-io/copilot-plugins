## pi-mobile

> - Preserve Android → authenticated WebSocket bridge → one isolated `pi --mode rpc` process per cwd.

# Pi Mobile contributor and agent guide

## Architecture invariants

- Preserve Android → authenticated WebSocket bridge → one isolated `pi --mode rpc` process per cwd.
- Do not migrate the bridge to the Pi SDK or remove cwd/session control locks.
- Pi 0.80.6 is the minimum tested runtime.
- Use documented RPC commands only. Active reads use `get_entries` and `get_tree`; the internal extension exists only for tree navigation because Pi 0.80.6 has no navigation RPC command.
- Unknown session entries trigger one explicit full rebuild. Never guess private session-file behavior.

## Modules

- `app`: Compose UI, screen state, host/token storage, and session controller.
- `core-rpc`: typed Pi commands/events and sanitized compatibility fixtures.
- `core-net`: WebSocket transport, authenticated bridge control, request correlation, reconnect, and entry cursors.
- `core-sessions`: cached session-index models and repository.
- `bridge`: authentication, WebSocket envelopes, locks, process lifecycle, inactive-session indexing, and internal extensions.
- `benchmark`: device-owned macrobenchmark/profile scaffolding.

Keep regression tests beside the owning module. Protocol fixtures belong in `core-rpc/src/test/resources/rpc`; they must be sanitized.

## Required commands

Use JDK 25 for Gradle and compilation, retain a JDK 21 toolchain for the stable detekt CLI, and install Android SDK platform 37.0/build-tools 37.0.0. Use Node 24 LTS+, pnpm 10, and Pi 0.80.6+.

```bash
./gradlew clean ktlintCheck detekt test :benchmark:compileBenchmarkKotlin :app:lintDebug :app:assembleDebug :app:assembleRelease
(cd bridge && pnpm install --frozen-lockfile && pnpm run check && pnpm audit --prod)
```

Compile device tests without launching a device:

```bash
./gradlew :app:compileDebugAndroidTestKotlin
```

Do not run connected/device acceptance unless the operator explicitly owns and requests it.

## Secrets

- Never print, render, log, commit, or copy tokens, authorization headers, `.env` contents, credentials, or private sessions.
- Tokens use Android Keystore AES-GCM and must not enter ordinary preferences or backup.
- Do not add signing credentials. Release assembly uses repository-safe unsigned/default configuration.

## Plan protocol

1. Read `plans/README.md`, the complete active plan, referenced docs, and current Pi docs.
2. Run the plan drift check and mark the row `IN PROGRESS` before editing.
3. Add characterization/regression tests with behavior changes.
4. Run focused checks, full plan gates, and `git diff --check`.
5. Use small Conventional Commits commits. Never push, merge, or rewrite history.
6. Mark a plan done only when non-device gates pass. Device-only validation is recorded as `PENDING — operator-owned` with executable steps and evidence fields.

---
> Source: [ayagmar/pi-mobile](https://github.com/ayagmar/pi-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
