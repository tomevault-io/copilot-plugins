## react-native-unity

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`@azesmway/react-native-unity` — a React Native native module (published to npm) that embeds a Unity project into RN as a full-fledged `<UnityView>` component. Supports both the **old architecture** (Paper/bridge) and the **new architecture** (Fabric/TurboModules) via React Native Codegen. This is a **library repo**, not an app; the app lives in `example/` (a yarn workspace used to test changes).

Package manager is **Yarn 3 (Berry)** with workspaces — always use `yarn`, not `npm`. Node >= 18 (`.nvmrc`).

## Commands

```sh
yarn                    # install deps for all workspaces
yarn typecheck          # tsc --noEmit (script is also `yarn typescript` per CONTRIBUTING)
yarn lint               # eslint; add --fix to autoformat (Prettier via eslint)
yarn test               # jest
yarn test path/to/file  # run a single test file
yarn prepare            # build the library with react-native-builder-bob -> lib/
yarn clean              # remove build artifacts (lib/, native build dirs)
yarn release            # release-it (version bump, tag, npm publish, GH release)

# Example app (from repo root)
yarn example start      # Metro bundler
yarn example android    # run example on Android (emulator or device)
yarn example ios        # run example on iOS device (NOT simulator — Unity crashes there)
```

Pre-commit hooks (lefthook + commitlint) run lint + tests and enforce **Conventional Commits** (`fix:`, `feat:`, `refactor:`, `docs:`, `test:`, `chore:`). Commit messages that don't match are rejected.

## Architecture

The plugin is one component with a JS shell, generated native bindings, hand-written native players, and an Expo config plugin.

### JavaScript layer (`src/`)
- `src/specs/UnityViewNativeComponent.ts` — the **Codegen spec**. Declares `NativeProps` and `NativeCommands` (`postMessage`, `unloadUnity`, `pauseUnity`, `resumeUnity`, `windowFocusChanged`) and exports the host component `RNUnityView` plus the `Commands` object. Codegen reads this file (`codegenConfig.name = "unityview"`, `jsSrcsDir = "src"`) to generate native interfaces for the new architecture. **Any new prop/command must be added here first**, then wired into all native targets.
- `src/UnityView.tsx` — public class component. Wraps the native component, exposes imperative methods (via `Commands.*`), and **unloads Unity on `componentWillUnmount`**.
- `src/index.ts` — re-exports `UnityView` as default.

### Native — Android (`android/src/`)
- `main/java/com/azesmwayreactnativeunity/` — the real implementation: `ReactNativeUnity` (holds the Unity player as a **static singleton** — one player process reused across mounts), `UPlayer`, `ReactNativeUnityView`, `ReactNativeUnityViewManager`, `ReactNativeUnityViewPackage`.
- `newarch/` and `oldarch/` — two versions of `ReactNativeUnityViewManagerSpec`. `android/build.gradle` selects the source set via `isNewArchitectureEnabled()` and adds the generated codegen sources only in new-arch builds.
- `main/AndroidManifest.xml` vs `AndroidManifestNew.xml` — arch-specific manifests.

### Native — iOS (`ios/`)
- `RNUnityView.{h,mm}` and `RNUnityViewManager.mm` — Objective-C++ view + manager.
- iOS does **not** embed a Unity source project. It consumes a **prebuilt `UnityFramework.framework`**. The podspec's `prepare_command` copies it from `unity/builds/ios/` into `ios/` and lists it as a `vendored_framework`. Old vs new arch is gated by `RCT_NEW_ARCH_ENABLED` in `react-native-unity.podspec`.
- `unity/Assets/Plugins/iOS/NativeCallProxy.{h,mm}` — the bridge Unity calls into to send messages back to RN.

### Expo config plugin (`plugin/`)
- `plugin/src/index.ts` — an `@expo/config-plugins` plugin that, for Expo/prebuild users, patches `android/build.gradle` (flatDir), `settings.gradle` (include `:unityLibrary` from `unity/builds/android`), `gradle.properties` (`unityStreamingAssets`), and `strings.xml`. Built to `plugin/build/` (its own `tsconfig.json`) and exposed via root `app.plugin.js`.

### The `unity/` folder
Scripts/assets the user copies into **their** Unity project. iOS build output goes to `unity/builds/ios` and Android to `unity/builds/android` in the consumer app — the plugin/podspec read from there.

## Key facts for making changes

- **Cross-cutting change rule**: a new prop or command touches four places — the Codegen spec (`src/specs/`), the JS wrapper (`src/UnityView.tsx`), Android (both `newarch`/`oldarch` specs + manager), and iOS (`RNUnityViewManager.mm`). Missing one breaks a platform or an architecture.
- Both RN architectures must keep working — don't remove the old-arch code paths.
- iOS runs on **device only**; the iOS simulator is unsupported (Unity/Metal crashes).
- On iOS the Unity view crashes (`MTLTextureDescriptor has width of zero`) if its RN parent has zero dimensions — views need real size (`flex: 1`).
- The library is consumed from `src` during example development (`react-native` field points to `src/index`), so JS changes hot-reload; native changes require a rebuild.

---
> Source: [azesmway/react-native-unity](https://github.com/azesmway/react-native-unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
