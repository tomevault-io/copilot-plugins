## homelab-project

> This repository is the single working copy for Homelab development.

# AGENTS.md - Homelab

This repository is the single working copy for Homelab development.

## Repository Flow

- Work from this repository root only.
- `origin` is the primary GitHub remote.
- `gitea` is an optional mirror remote only.
- Do not use the old Gitea working copy for new work.
- Keep `main` tracking `origin/main`.

## Branch Strategy

- Use `main` for normal owner-directed changes, release preparation, and release follow-up commits.
- Create a short-lived branch for larger/riskier work, external PR review, or changes that should not block release work.
- Prefer branch names like `feat/service-name`, `fix/issue-name`, `docs/topic`, or `ci/topic`.
- Use a separate git worktree only when another uncommitted task is already in progress and switching branches would risk mixing changes.
- Do not merge or close external PRs without reviewing the diff and running the relevant checks.

## Development Rules

- Make focused commits with clear messages:
  - `feat: ...`
  - `fix: ...`
  - `docs: ...`
  - `ci: ...`
  - `chore: ...`
- For release-bound changes, update both platform versions together:
  - Android: `HomelabAndroid/app/build.gradle.kts`
  - iOS: `HomelabSwift/Homelab/Info.plist`
- Keep Android `versionCode` and iOS `CFBundleVersion` aligned.
- Keep Android `versionName` and iOS `CFBundleShortVersionString` aligned.
- Do not commit generated release binaries (`.ipa`, `.apk`, `.aab`) unless explicitly requested.

## Verification Policy

- Run local checks based on the files touched; do not run every build for every change by default.
- Docs-only changes (`README.md`, `AGENTS.md`, license, markdown, screenshots) do not require local Android or iOS builds.
- Android-only code/resources require the Android compile check; run Android unit tests when logic, networking, parsing, storage, or ViewModels change.
- iOS-only code/resources require the iOS compile check; run iOS unit tests when logic, networking, parsing, storage, or model behavior changes.
- Cross-platform service changes, shared release metadata, or version bumps require both Android and iOS compile checks.
- Release publishing with user-provided signed `Homelab.ipa` and `Homelab.apk` does not require rebuilding locally unless source code changed in the same task.
- After pushing to `main`, always inspect the GitHub Actions `CI` run. The task is not complete if CI fails.

## Build Checks

Android compile check:

```bash
cd HomelabAndroid
GRADLE_USER_HOME="$PWD/.gradle-home" \
JAVA_HOME=$(/usr/libexec/java_home -v 21) \
./gradlew :app:compileDebugKotlin --console=plain
```

iOS compile check without launching simulators:

```bash
cd HomelabSwift
xcodebuild build \
  -project Homelab.xcodeproj \
  -scheme Homelab \
  -configuration Debug \
  -sdk iphoneos \
  -destination 'generic/platform=iOS' \
  -derivedDataPath /private/tmp/homelab-ios-dd \
  CODE_SIGNING_ALLOWED=NO
```

## Test Checks

Android unit tests:

```bash
cd HomelabAndroid
GRADLE_USER_HOME="$PWD/.gradle-home" \
JAVA_HOME=$(/usr/libexec/java_home -v 21) \
./gradlew :app:testDebugUnitTest --console=plain
```

iOS unit tests require an available iOS simulator:

```bash
cd HomelabSwift
xcodebuild test \
  -project Homelab.xcodeproj \
  -scheme Homelab \
  -configuration Debug \
  -destination 'platform=iOS Simulator,name=iPhone 17' \
  -derivedDataPath /private/tmp/homelab-ios-test
```

If the exact simulator is unavailable, list devices with `xcrun simctl list devices available` and adjust the destination.

## Release Flow

- Release builds are manual unless signing automation is explicitly added later.
- The user provides the final signed `Homelab.ipa` and `Homelab.apk`.
- Create a GitHub release with tag `vX.Y.Z` and upload both assets.
- The `Update AltStore Source` workflow updates `apps.json` and `app-version.json`.
- After the workflow succeeds, pull `origin/main`.
- Push `main` to `gitea` only as a mirror if desired.

## Manifest Rules

- Do not manually edit `apps.json` or `app-version.json` for normal releases.
- The release workflow extracts iOS build metadata from the uploaded IPA.
- Always verify after a release:
  - `app-version.json.latest`
  - `apps.json` latest version entry
  - IPA/APK URLs point to the new release
  - GitHub Actions run status is `success`

## Platform Notes

- SideStore and AltStore Classic/World are supported through the IPA source.
- Keep README wording specific: use "AltStore Classic / SideStore" when discussing sideloading.

---
> Source: [JohnnWi/homelab-project](https://github.com/JohnnWi/homelab-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
