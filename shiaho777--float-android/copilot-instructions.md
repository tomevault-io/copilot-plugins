## float-android

> Instructions for coding agents working in this repository.

# AGENTS.md

Instructions for coding agents working in this repository.

## Code style

- Do what you believe is right. Make the change complete and correct, not the smallest possible diff. If a fix calls for refactoring, renaming, or touching multiple files, do it.
- Fix root causes, not symptoms. Do not paper over a defect with catch-and-swallow, degraded fallbacks, or UI workarounds that leave the underlying behavior broken. Trace the failure to where it originates and repair it there — even when that is harder.
- Match the patterns and conventions already in the surrounding code.
- Do not add copyright or license headers unless asked.

## Project layout

| Path | Role |
|------|------|
| `entries/` | Vite multi-page entry points: `main.tsx` (phone), `world-builder.tsx` (3D world builder), `characters.tsx` (standalone character page). Each maps to an HTML file at repo root. |
| `components/` | React UI tree — chat, desktop, settings, check-phone simulated apps, findmy, moments, games, etc. (~6.8 MB, dense codebase; read neighbors before adding). |
| `lib/` | Business logic layer (~268 modules): Dexie/IndexedDB stores, LLM adapter, prompt assembler, engines (chat/group-chat/memory/presence/dwelling/checkphone), importer, backup. |
| `styles/components.css` | Single design-token CSS file shared by all components (`dl-*`, `ap-*`, `fi-*`, `cp-*` families). Reuse existing classes before inventing new ones. |
| `android/` | Capacitor Android project. Native plugins/services live in `android/app/src/main/java/app/floatphone/app/`. |
| `custom-apps/` | Built-in user apps installed through the Custom App SDK. |
| `app-store-apps/` | Extra apps for the in-app market. |
| `world-builder/` | 3D world builder page (React Three Fiber). |
| `characters/` | Standalone character management page. |
| `docs/` | Specs and plans (docs/specs/, docs/plans/) plus README screenshots in `docs/screenshots/`. |

User-facing product doc: `README.md` (Chinese-first). The upstream project this repo is built from is `xiaolongbao0709/ai-virtual-phone` — see the README acknowledgement section.

## How the main pieces connect

```text
Vite multi-page build (vite.config.ts)
  index.html / world-builder/index.html / characters/index.html
  → entries/*.tsx → components/* + lib/*
  → out/ → capacitor.config.ts (webDir: out, scheme: https)
  → npx cap sync android → Android WebView shell (MainActivity)

WebView page
  → Capacitor bridge → native plugins (Java/Kotlin under android/app/.../app/floatphone/app/)
  → Dexie/IndexedDB (all app data, per-entity tables)
  → user-configured third-party APIs (LLM / image / voice / music / map tiles)
```

Mental model:

- **Everything is client-side.** There is no project backend. Data lives in IndexedDB (via Dexie stores in `lib/`), app-private native media storage, and localStorage. Supabase-dependent modules from upstream were removed in this branch; direct third-party connectivity (LLM, image gen, music, map tiles) remains.
- **A feature that must survive screen-off / app-switch** must route through the keep-alive service (`GenerationKeepAliveService` + plugin) and use native HTTP/SSE (`NativeHttpPlugin` via `lib/native-http.ts`) instead of `fetch`, so streaming survives WebView throttling.
- **Large media must not travel over the bridge as base64.** Import/export writes chunks via `Filesystem` (`lib/download-utils.ts`); media blobs are stored natively and referenced as `media-store://<id>` (`NativeMediaPlugin`, `lib/native-media.ts`).

## Platform constraints

- **CORS:** GitHub release assets, some API endpoints, and arbitrary user-hosted files lack CORS headers — WebView `fetch`/`XHR` fails on them even though the network is fine. Route such requests through `httpFetch`/`nativeHttp` (OkHttp) or Dexie's own sync — never assume browser `fetch` reaches them.
- **Memory:** never accumulate large base64 strings in JS for import/export; the bridge serializes every call. Use the chunked `writeFile`/`appendFile` pattern in `lib/download-utils.ts` (16 MB chunks).
- **Bridge payloads:** Capacitor bridge messages are JSON — sending MB-scale progress/blob data per event floods it. Native plugins emit throttled progress events only; bytes stay native-side.
- **`targetSdk` 35 storage:** public-Documents access requires `MANAGE_EXTERNAL_STORAGE` via `StorageAccessPlugin` → system settings grant → app restart. `lib/storage-access.ts` drives the prompt flow; `lib/auto-backup.ts` and exporters must go through `StorageAccessPlugin.requirePermission()` before touching `/storage/emulated/0/`.
- **Self-update:** `lib/app-updater.ts` + `AppUpdaterPlugin` implement check/download/install against GitHub Releases. Downloads are native OkHttp streams with Range-resume; the JS layer is a state machine only. Progress events are only accepted while state is `downloading` (pause-race guard) — keep that invariant when touching it.
- **Version alignment:** `package.json` version, `android/app/build.gradle` `versionName`, and the release tag must agree (`1.0.0` ↔ `v1.0.0`). The updater compares `versionName` against tag names.
- Avoid new third-party dependencies unless strongly justified; prefer existing modules in `lib/` and platform APIs.

## Workflow

- Do not commit secrets, `local.properties`, `android/keystore.properties`, keystores (`*.keystore`), or IDE/cache junk. Signing material is gitignored — keep it that way.
- Do not create commits, push, open PRs, or file Issues unless the user asks to deliver / ship / push / open a PR (or equivalent).
- Verify before handing off: `npx tsc --noEmit` (strict; repo should stay at 0 errors), `npm run build`, and `./gradlew assembleDebug` or `assembleRelease` inside `android/` when native code changed. Emulator verification is preferred for bridge-level changes.

### Delivery (Issue + PR + CI)

Default target: [shiaho777/float-android](https://github.com/shiaho777/float-android). Prefer a pull request over direct pushes to `main` when delivering code.

**Language (required):** GitHub **Issues and PRs must be written in English** — titles, bodies, labels text you author, and delivery comments on the Issue/PR. Local chat with the user may be Chinese or any language; do not copy that language into Issue/PR text.

When the user asks to deliver a change, run the Issue → branch → PR → CI → merge loop end-to-end. Do not close the Issue until the PR is merged and CI is green.

**Branch naming:** use plain `type/slug` names — `fix/…`, `feat/…`, `refactor/…`, `docs/…`, `perf/…`, `chore/…`. Do not use tool/agent namespaces (`codex/…`, `devin/…`, etc.); the branch belongs to the repo, not the agent.

PR bodies follow `.github/pull_request_template.md` and must include `Fixes #N` (or `Closes #N`) so the Issue closes on merge — never on PR open, never while checks are red.

### Releases

Releases are GitHub Releases built from `main`:

1. Bump `versionName` in `android/app/build.gradle` and `version` in `package.json` to the new `X.Y.Z`; bump `versionCode` by 1 (Android treats upgrades by `versionCode`, the updater compares `versionName`).
2. Build: `npm run build` → `npx cap sync android` → `cd android && ./gradlew assembleRelease` (signed via local `keystore.properties`, never committed).
3. Publish: `gh release create vX.Y.Z float-android-X.Y.Z.APK --repo shiaho777/float-android --target main --title "Float vX.Y.Z" --notes "…"` — asset name is `float-android-X.Y.Z.APK`, tag is `vX.Y.Z` on `main`.
4. To re-spin the same version (hot-fixing a just-published release), delete and recreate the release+tag at the new commit rather than pushing a moved tag silently: `gh release delete vX.Y.Z --cleanup-tag` then the same `gh release create` line.

The in-app updater (设置 → 关于与声明) lists these releases and downloads the first `*.apk` asset — keep exactly one APK asset per release.

---
> Source: [shiaho777/float-android](https://github.com/shiaho777/float-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
