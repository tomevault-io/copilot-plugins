## tracelet

> **NEVER run `dart pub publish` manually.** All publishing is done via the GitHub Actions workflow.


# Publishing Instructions

## CRITICAL — Do Not Manually Publish

**NEVER run `dart pub publish` manually.** All publishing is done via the GitHub Actions workflow.

### How to Publish
1. Bump versions and update changelogs locally (see checklists below)
2. Commit and push to `main`
3. Go to GitHub → Actions → **"Release (Native SDKs + pub.dev)"** → **Run workflow**
4. Options:
   - `dry_run`: Build + lint only, publish nothing (use first to verify)
   - `skip_native_sdks`: Skip Maven Central + CocoaPods (if native SDKs unchanged)
   - `skip_flutter`: Skip pub.dev (publish only native SDKs)

### Why Not Manual?
- Manual publish risks **partial releases** (some packages published, others not)
- pub.dev versions are **immutable** — a partially published version cannot be fixed, only bumped
- The workflow handles: validation → native SDKs (parallel) → Flutter packages (sequential with 30s indexing delays) → tagging
- The workflow checks if versions are already published and skips them (idempotent)

### If You Accidentally Publish Manually
If some packages were published at version X.Y.Z but not all:
1. Bump ALL packages to X.Y.(Z+1)
2. Add changelog entry: `**CHORE**: Re-release — X.Y.Z was partially published without all fixes.`
3. Push and trigger the release workflow
4. The workflow will skip already-published native SDKs (Maven Central / CocoaPods check)

## Three Distribution Channels
| Channel | Artifact | Registry |
|---------|----------|----------|
| Android SDK | `com.ikolvi:tracelet-sdk` | Maven Central |
| iOS SDK | `TraceletSDK` | GitHub Release (Bundled) |
| Flutter | 8 federated packages | pub.dev |

Native SDKs version independently from Flutter. Flutter packages are always version-locked together.

## Publishing Order (STRICT)
1. **Native SDKs** (independent, parallel OK): Android → Maven Central, iOS → GitHub Release (Bundled)
2. **Flutter packages** (sequential, wait for each to appear on pub.dev):
   1. `tracelet_platform_interface` (no Tracelet deps)
   2. `tracelet_android` (depends on interface)
   3. `tracelet_ios` (depends on interface)
   4. `tracelet_web` (depends on interface)
   5. `tracelet` (depends on all above)
   6. `tracelet_sync` (optional, depends on tracelet)
   7. `tracelet_supabase` (optional, depends on tracelet)
   8. `tracelet_firebase` (optional, depends on tracelet)
   9. `tracelet_doctor` (optional, depends on tracelet)

pub.dev resolves deps at publish time — a package cannot reference a version that doesn't exist yet.

## Pre-Release Checklist
### Flutter (all core packages must match version)
- Bump `version:` in all 5 core `packages/*/pubspec.yaml`
- Bump `version:` in `packages/tracelet_sync/pubspec.yaml`, `packages/tracelet_supabase/pubspec.yaml`, `packages/tracelet_firebase/pubspec.yaml`, and `packages/tracelet_doctor/pubspec.yaml` (if publishing them)
- Update cross-package `^X.Y.Z` constraints (see below)
- Add entries to all relevant `packages/*/CHANGELOG.md` with `**FEAT**:`/`**FIX**:`/`**PERF**:` prefixes
- Bump native library dependency version inside wrappers:
  - Android: `packages/tracelet_android/android/build.gradle` → `implementation("com.ikolvi:tracelet-sdk:X.Y.Z")`
  - iOS: `packages/tracelet_ios/ios/tracelet_ios.podspec` → `s.dependency 'TraceletSDK', 'X.Y.Z'`
  - Sync Android: `packages/tracelet_sync/android/build.gradle.kts` → `implementation("com.ikolvi:tracelet-sync-sdk:X.Y.Z")` and `compileOnly("com.ikolvi:tracelet-sdk:X.Y.Z")`
  - Sync iOS: `packages/tracelet_sync/ios/tracelet_sync.podspec` → `s.dependency 'TraceletSDK', 'X.Y.Z'`

### Cross-Package Dependency Constraints
When publishing version X.Y.Z, ALL constraints must point to the version being published:
```yaml
# tracelet_android/pubspec.yaml, tracelet_ios/pubspec.yaml, tracelet_web/pubspec.yaml
tracelet_platform_interface: ^X.Y.Z

# tracelet/pubspec.yaml
tracelet_platform_interface: ^X.Y.Z
tracelet_android: ^X.Y.Z
tracelet_ios: ^X.Y.Z
tracelet_web: ^X.Y.Z

# tracelet_sync/pubspec.yaml, tracelet_supabase/pubspec.yaml, tracelet_firebase/pubspec.yaml, tracelet_doctor/pubspec.yaml
tracelet: ^X.Y.Z

# tracelet_supabase/pubspec.yaml, tracelet_firebase/pubspec.yaml
tracelet_sync: ^X.Y.Z
```
Never publish with stale constraints pointing to older versions.

### Native SDKs (only if changed)
- Android: `sdk/android/gradle.properties` → `SDK_VERSION`, `sdk/android/CHANGELOG.md`
- iOS:
  - Root: `TraceletSDK.podspec` → `s.version = 'X.Y.Z'`
  - Sub: `sdk/ios/TraceletSDK.podspec` → `s.version = 'X.Y.Z'`
  - Changelog: `sdk/ios/CHANGELOG.md`

## Git Tags
Git Tags are no longer manually created. The automated release workflow skips creating tags for Flutter packages to prevent discrepancies. Do not run any commands (like `melos version`) that auto-generate git tags.

## Automated Release
Trigger via GitHub Actions: `.github/workflows/release.yml` → Run workflow.
Options: `dry_run`, `skip_native_sdks`, `skip_flutter`, `publish_tracelet_doctor`, `publish_tracelet_sync`, `publish_tracelet_supabase`, `publish_tracelet_firebase`.

## Manual Flutter Publish (EMERGENCY ONLY — prefer GitHub Actions workflow)
```bash
# Only use if GitHub Actions is down. Risk of partial publish!
# Publish in strict order — wait 30s between each for pub.dev indexing
cd packages/tracelet_platform_interface && dart pub publish --force
cd packages/tracelet_android && dart pub publish --force
cd packages/tracelet_ios && dart pub publish --force
cd packages/tracelet_web && dart pub publish --force
cd packages/tracelet && dart pub publish --force
cd packages/tracelet_sync && dart pub publish --force # Optional
cd packages/tracelet_supabase && dart pub publish --force # Optional
cd packages/tracelet_firebase && dart pub publish --force # Optional
cd packages/tracelet_doctor && dart pub publish --force # Optional
git push origin --tags
```

## Key File Locations
| What | Path |
|------|------|
| Flutter versions | `packages/*/pubspec.yaml` |
| Flutter changelogs | `packages/*/CHANGELOG.md` |
| Android SDK version | `sdk/android/gradle.properties` |
| iOS SDK version | `TraceletSDK.podspec` |
| tracelet_sync native deps | `packages/tracelet_sync/android/build.gradle.kts`, `packages/tracelet_sync/ios/tracelet_sync.podspec` |
| Release CI | `.github/workflows/release.yml` |

## Troubleshooting
- **pub.dev dep resolution fails**: Publish in order; `^X.Y.Z` must reference already-published versions.
- **Maven Central stuck**: Run `closeAndReleaseSonatypeStagingRepository` separately.
- **CocoaPods 409**: Version exists — bump `s.version`.

## Quality Verification with Melos

To verify and automatically apply code formatting across all packages before release:

```bash
# Fix and apply formatting across all packages
dart run melos run format:fix

# Verify that formatting is completely correct
dart run melos run format
```

## Native Binary Artifacts & Codegen
- **iOS XCFramework**: Pushed as a `.zip` file directly to the **GitHub Release** (created by the `publish-ios-sdk` job). CocoaPods downloads this zip via the URL in the `.podspec`.
- **Android AAR**: The compiled Rust libraries (`.so` files) are packaged into the `.aar` file. This `.aar` is pushed directly to **Maven Central** (created by the `publish-android-sdk` job). There is no zip on GitHub Releases for Android.
- **FRB Codegen**: The `frb_generated.rs` and other flutter_rust_bridge bindings are intentionally **committed to Git**. Native developers do not need Flutter installed. 
- **CI Codegen Enforcement**: The CI validation workflows (`ci.yml` and `release.yml`) run `flutter_rust_bridge_codegen generate` and enforce parity via `git diff`. If bindings are out of sync, CI fails.

---
> Source: [Ikolvi/Tracelet](https://github.com/Ikolvi/Tracelet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
