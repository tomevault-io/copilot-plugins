## react-native-pose-landmarks

> - This is a single-package RN Nitro module repo (`react-native-pose-landmarks`) with one workspace app at `example/`.

# AGENTS

## Repo shape (do not guess)
- This is a single-package RN Nitro module repo (`react-native-pose-landmarks`) with one workspace app at `example/`.
- Public JS entrypoint is `src/index.ts`; Nitro spec is `src/specs/pose-landmarks.nitro.ts`.
- Native implementations live in `android/src/main/java/com/margelo/nitro/poselandmarks/` and `ios/`.
- `nitrogen/generated/**` is tracked generated code (not throwaway build output).
- Android example bundles the MediaPipe model at `example/android/app/src/main/assets/pose_landmarker_lite.task`.

## Commands that are source-of-truth
- Install deps from repo root with `npm install` (lockfile is `package-lock.json`).
- Library checks/build: `npm run typecheck` then `npm run build`.
- Regenerate Nitro bindings after spec/native interface changes: `npm run codegen`.
- Run example app from root via workspace scripts:
  - Android: `npm run -w example android`
  - iOS: `npm run -w example ios`
  - Metro: `npm run -w example start`
  - Pods: `npm run -w example pod`

## Codegen + native quirks
- `npm run codegen` runs `nitrogen`, then `bob build`, then `node post-script.js`.
- `post-script.js` patches `nitrogen/generated/android/PoseLandmarksOnLoad.cpp` (removes `margelo/nitro/` include prefix). If that file reverts after regeneration, re-run codegen before committing.
- Keep the Android wrapper class `android/src/main/java/com/poselandmarks/HybridPoseLandmarks.kt`; it bridges package name expectations for Nitro registration.
- Current Android defaults are `MODEL_POSE_LANDMARKER_LITE` and `DELEGATE_CPU` in `PoseLandmarkerHelper` for compatibility and startup stability.
- `PoseLandmarkerHelper` now checks whether the selected model exists in app assets before creating the landmarker and surfaces a listener error when missing.

## Build/CI realities
- Actual Android app build target is `example/android` (see CI); root `build.gradle`/`settings.gradle` are for IDE/LSP sync support.
- CI Android workflow currently uses `bun install`, but the repo is npm-managed; prefer npm locally unless you are explicitly reproducing CI.
- iOS CI workflow file exists but is currently empty (`.github/workflows/ios-build.yml`). Do not assume automated iOS validation exists.

## Verification expectations for edits
- JS/TS-only edits: run `npm run typecheck` and `npm run build`.
- Spec or native signature edits: run `npm run codegen` first, then `npm run typecheck` and `npm run build`.
- Native runtime edits: also build the example target you touched (`npm run -w example android` or `npm run -w example ios`).

## Troubleshooting quick map
- `initPoseLandmarker()` returns `false` on Android: check logcat for `Pose Landmarker model asset ... not found` and verify the `.task` model is in app assets.
- Landmarks stay empty: verify camera permission is granted in the host app and that camera binding succeeded (look for `bound imageAnalyzer to lifecycle successfully`).
- Regenerated files include path issues: run `npm run codegen` to apply the post-generation include patch.

## Version/release notes
- Tag-based GitHub release workflow publishes on `v*` tags (`.github/workflows/release.yml`).
- Commit style matters for semantic-release config (`feat`, `fix`, `perf`, `refactor`, `docs`, `chore` in `release.config.cjs`).
- Prefer release notes that call out runtime behavior changes (initialization failures, asset requirements, defaults, and troubleshooting hints).

---
> Source: [tony-div/react-native-pose-landmarks](https://github.com/tony-div/react-native-pose-landmarks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
