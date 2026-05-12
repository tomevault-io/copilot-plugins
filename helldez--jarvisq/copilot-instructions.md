## jarvisq

> handles downloads; HTTP fallback lives in `downloadTtsWithCompanions` and

# Contributor & agent guide

This file is the contract for both human contributors and AI coding agents
working on the JarvisQ codebase. Read it fully before touching files.

## Project in one paragraph

JarvisQ is an Expo + React Native consumer of the Tether **`@qvac/sdk`**
that ships an on-device voice assistant (STT → LLM → TTS). The codebase
follows hexagonal / ports-and-adapters architecture so the same core can power
mobile today and desktop (Windows / macOS / Linux) tomorrow. See `README.md`
for the full architecture map.

## Non-negotiable rules

1. **Do not fork or wrap `@qvac/sdk`.** It is consumed directly from
   `node_modules`. Upgrading must remain a `package.json` bump — never add a
   compatibility shim that requires maintenance on SDK updates. The one
   sanctioned exception is `patches/@qvac+sdk+<version>.patch`, applied
   automatically via `patch-package` in `postinstall`; when bumping the SDK,
   refresh the patch (or drop it if upstream fixed the issue) rather than
   introducing wrapper code.
2. **`src/core/` is platform-free.** No `expo-*`, `react-native-*`,
   `@react-native-*`, Node-only, or Bare-only imports are permitted in
   `src/core/`. New capabilities enter the core only through a port under
   `src/core/ports/`; the concrete implementation lives in
   `src/platform/<target>/`.
3. **No hardcoded paths or constants at call sites.** Paths derive from
   `IFileSystem.documentDirectory` and `AppConfig`. Model URLs live in
   `src/core/config/HttpModelSources.ts`; model selection lives in
   `src/core/config/ModelConfig.ts`.
4. **English code and comments only.** Identifiers, comments, commit
   messages, PR descriptions — all English. User-facing UI copy is the only
   exception.
5. **Android works and must keep working.** Verify with `npm run typecheck`
   at minimum. Do not remove the Expo mobile adapters or the
   `bootstrapMobile()` wire-up.
6. **Additive refactors.** When introducing a port, keep the existing public
   call surface stable where practical so feature work and infra work can be
   reviewed independently.

## Where to add things

| Change | Location |
|--------|----------|
| New on-device capability (e.g. embeddings) | `src/core/inference/` + a port if it has platform-specific parts |
| New persisted data | `src/data/repositories/` (uses `IDatabase`) |
| New user setting | `src/domain/SettingsStore.ts` + `src/domain/types.ts` |
| New screen | `src/app/` (Expo Router file) |
| New shared UI primitive | `src/ui/components/` |
| New platform target | new folder under `src/platform/<target>/` with adapters + `bootstrap.ts` |
| New HTTP model URL | `src/core/config/HttpModelSources.ts` (pin a commit SHA) |

## Ports cheat-sheet

The following live under `src/core/ports/`. Keep their surface minimal — only
add methods when a real consumer needs them.

- `IAudioRecorder` — microphone + VAD.
- `IAudioPlayer` — PCM output.
- `IFileSystem` — `getInfo`, `makeDirectory`, `move`, `download`,
  `documentDirectory`.
- `IKeyValueStore` — `getItem` / `setItem` / `removeItem`.
- `IDatabase` — `exec`, `run`, `getFirst`, `getAll`.
- `IHaptics` — short tactile feedback for UI events.
- `IPermissions` — runtime permission checks/requests (microphone today).
- `INetworkInfo` — reachability probe used by `loadWithFallback` to decide
  between P2P registry and HTTPS fallback. A fetch-based default lives in
  `src/core/net/FetchNetworkInfo.ts` and is reused by both mobile and desktop.

### TTS engine contract (`ITtsService`)

Unlike the other ports, the TTS service is not pinned to one implementation
at startup — `VoicePipeline` receives the concrete `ITtsService` via
`PipelineDeps` and the app chooses between `TtsService` (Supertonic) and
`SystemTtsService` (`expo-speech`) based on `AppSettings.ttsEngine`. The
interface is intentionally play-oriented (not synthesize-then-play):

- `speak(text, audioPlayer, options?)` — plays text and resolves on
  completion. PCM engines (Supertonic) write into `audioPlayer`; native
  engines (System) ignore it and manage their own playback.
- `stop()` — aborts current playback without unloading.

Do not leak `synthesize() → Float32Array` back onto this port: native
engines cannot return PCM, and adding it would force branching in
`VoicePipeline.drainTtsQueue`.

Resolve adapters at runtime through `getPlatform()` from
`@core/platform/PlatformContainer` — never `new ExpoX()` from a core file.

## Verification before committing

```bash
npm run typecheck
```

If UI changed, manually validate on an Android device; automated UI tests do
not exist yet. See the **Android release build & verify** section below for
the standalone-APK workflow that survives USB disconnect.

## Android release build & verify (standalone APK)

Standard incremental rebuild — Gradle picks up JS/RN changes automatically
through `createBundleReleaseJsAndAssets` once the source tree changes:

```bash
cd android && ./gradlew assembleRelease
adb install -r android/app/build/outputs/apk/release/app-release.apk
```

The APK is **standalone**: it embeds the JS bundle (`assets/index.android.bundle`,
Hermes bytecode) so the app keeps working after USB disconnect — no Metro
dependency. Do **not** use `npx expo run:android --variant release` as the only
check; it can leave Metro listening on localhost and mask bundle-freshness
issues.

### Native-config changes require prebuild

If you touch `app.json`, install a new Expo plugin, or change anything that
affects `AndroidManifest.xml` / native Gradle config, you MUST regenerate the
native project first:

```bash
npx expo prebuild --platform android
```

Skipping this is the most common cause of "I rebuilt but my change isn't in
the APK." Typical triggers: adding `expo-splash-screen`, changing `package`,
editing splash/icon assets, changing permissions.

### Verifying the APK actually contains your changes

`FROM-CACHE` / `UP-TO-DATE` in the Gradle log does **not** mean your JS
changes shipped — it means Gradle believed the cache key matched. The
authoritative check is to grep the embedded Hermes bundle for a unique
string from your change:

```bash
unzip -p android/app/build/outputs/apk/release/app-release.apk \
  assets/index.android.bundle > /tmp/b.hbc
grep -c -a "SOME_UNIQUE_STRING_FROM_YOUR_EDIT" /tmp/b.hbc   # must be > 0
```

If the count is 0, the bundle is stale. Clean and retry:

```bash
rm -rf android/app/build android/app/.cxx android/build
cd android && ./gradlew assembleRelease
```

### Troubleshooting "I see the old UI"

Checklist, in order, before blaming cache:

1. **Screen state** — `adb shell dumpsys power | grep mWakefulness`. If
   `Dozing`, the screenshot will be solid black. Wake with
   `adb shell input keyevent KEYCODE_WAKEUP`.
2. **Foreground window** — `adb shell dumpsys window | grep mCurrentFocus`.
   If it says `NotificationShade` or another app, the JarvisQ UI isn't
   actually visible.
3. **Fresh bundle** — verify with the `grep -c -a` trick above.
4. **Splash screen stuck** — if a centered icon floats mid-screen over the
   real UI, `expo-splash-screen` isn't hiding. We call
   `SplashScreen.preventAutoHideAsync()` + `hideAsync()` in
   `src/app/_layout.tsx`; don't remove it.
5. **Two apps installed** — `adb shell pm list packages | grep jarvis`. The
   current package is `app.jarvisq.mobile`. A legacy
   `com.anonymous.jarvisqvac` or `com.jarvis.assistant` may coexist after
   the rename; the launcher icon is ambiguous.

Only after 1–5 check out should you reach for a full clean
(`adb uninstall app.jarvisq.mobile` + wipe `android/app/build`).

## Desktop (Windows) build & verify

The desktop target is an Electron shell that hosts `VoicePipeline` in the main
process and uses the renderer (Chromium) for Web Audio capture + playback.
The `@qvac/sdk` node RPC client auto-locates the Bare worker at
`resources/app.asar.unpacked/qvac/worker.entry.mjs` when packaged — the worker
is the pre-generated `qvac/worker.entry.mjs` checked into the repo.

### First-time setup

```bash
npm install --legacy-peer-deps
npm run desktop:bundle-worker   # regenerate qvac/worker.entry.mjs if plugins changed
```

### Dev loop

```bash
npm run desktop:dev             # builds build/desktop then launches Electron
```

The dev script runs `scripts/build-desktop.mjs` (esbuild) and starts Electron
against `build/desktop/main.mjs`. No Metro / hot-reload — edit then re-run.

### Windows installer

```bash
npm run desktop:dist:win        # produces release/desktop/win/JarvisQ-<v>-x64.exe
```

`electron-builder.json` emits a per-user NSIS installer. `qvac/**` is copied
into `resources/qvac/` as `extraResources`, and `node_modules/@qvac/**` and
`node_modules/bare-*/**` are `asarUnpack`ed so the SDK's Bare worker can load
its native addons from the filesystem.

### Do NOT on desktop

- Do not import Expo / React Native modules from `electron/**` or anything in
  the main-process graph. The build script stubs a handful of mobile-only
  modules (`react-native`, `expo-modules-core`, `modules/device-perf/src`)
  through an esbuild plugin — expanding that stub list is a smell. Keep
  platform-specific code in `src/platform/<target>/` and consume it through
  ports.
- Do not hardcode the app-data directory. Use
  `@platform/desktop/paths.ts → getAppDataDirectory() / getCacheDirectory()`.
- Do not spawn the Bare worker manually. `node-rpc-client` handles discovery,
  spawning, and RPC — see `QVAC_WORKER_PATH` if you need to override.

### Verifying the installer actually packages the worker

After `desktop:dist:win`, inspect the installer payload:

```bash
7z l release/desktop/win/JarvisQ-*.exe | grep -E "worker.entry|addons.manifest"
```

Both `qvac/worker.entry.mjs` and `qvac/addons.manifest.json` must appear. If
either is missing, the SDK will silently fall back to the stock worker and
miss any custom plugin configured in `qvac.config.json`.

## Commit style

Conventional Commits. Use `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`.
Subject under 70 chars. Body explains **why**, not what.

## What NOT to do

- Do not add generic "utility" files in `src/core/utils/` that reach into
  platform APIs — that is how leaks start.
- Do not reintroduce `ModelDownloader` / `ModelStorage`: the SDK's registry
  handles downloads; HTTP fallback lives in `downloadTtsWithCompanions` and
  in per-service fallback paths.
- Do not swap singletons for services that should be request-scoped. Pass
  adapters in via method arguments (e.g. `TtsService.load(config, { fileSystem })`)
  when the dependency is only needed for that call.
- Do not silently catch errors to hide symptoms. Fix root causes; surface
  unexpected failures.
- Do not hardcode a TTS language whitelist. The `system` engine picks up
  whatever language / voice / engine the user has configured in Android's
  system TTS settings. The Settings screen exposes a shortcut that opens
  `com.android.settings.TTS_SETTINGS`; leave that flow in place.
- Do not call `tts.synthesize(...)` / `audioPlayer.addChunk(...)` from
  `VoicePipeline`. Always go through `tts.speak(clause, audioPlayer, options)`
  so the system TTS adapter stays interchangeable.

---
> Source: [Helldez/JarvisQ](https://github.com/Helldez/JarvisQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
