## flutter-store-readiness-skill

> >


## Parameters

The skill accepts an optional comma-separated list of category IDs to audit only specific checks.

| Argument | Example | Effect |
|---|---|---|
| _(none)_ | `/flutter-store-readiness` | Full audit — all 8 checks |
| Single category | `/flutter-store-readiness privacy-manifest` | Only iOS Privacy Manifest |
| Multiple categories | `/flutter-store-readiness metadata,screenshots,age-rating` | Only listed checks |

**Valid category IDs:**

| ID | What it checks |
|----|----------------|
| `privacy-manifest` | iOS PrivacyInfo.xcprivacy completeness and required reason APIs |
| `data-safety` | Android Data Safety section and privacy policy |
| `permissions` | Permission usage descriptions and justification |
| `screenshots` | Screenshot sizes, app icons, and store assets |
| `age-rating` | Age rating, content descriptors, COPPA compliance |
| `metadata` | App name, description, keywords — length limits and requirements |
| `ios` | iOS-specific: Info.plist, export compliance, encryption, entitlements |
| `android` | Android-specific: targetSdk, signing, bundle format, ProGuard |

If arguments are provided, skip all unlisted categories. The report header notes which categories were in scope.

---

# App Store Submission Readiness Skill

You are performing a structured pre-submission checklist audit on a Flutter project for
both the Apple App Store and Google Play Store. Your job is to read config files, manifests,
assets, and source code to find issues that will cause rejection or require a resubmission.

Before starting, read the detailed check patterns and requirements in:
`references/store_checks.md`

---

## Process

### Step 1 — Understand the project layout

Start with a high-level read before any searching:

- List the root directory: identify `ios/`, `android/`, `pubspec.yaml`, `assets/`
- In `ios/`: look for `Runner/Info.plist`, `Runner/PrivacyInfo.xcprivacy`, `Runner.xcodeproj/`, `Podfile`
- In `android/`: look for `app/src/main/AndroidManifest.xml`, `app/build.gradle` (or `build.gradle.kts`), `key.properties`, `proguard-rules.pro`
- Note the app's domain (fintech, health, social, kids?) — this affects severity ratings for age rating and privacy checks

### Step 2 — Run all 8 audit categories

Work through every category in order (or only the ones requested via parameters). For each:

1. **Read** the relevant files using the patterns in `references/store_checks.md`
2. **Check** each item in the category checklist
3. **Assess severity** — will this cause a **rejection** (Critical), **warning** (High), or is it a **best-practice gap** (Medium/Low)?
4. **Write a fix** — one concrete, actionable sentence

Every category produces a ✅ / ❌ / ⚠️ checklist, not just a findings list.

---

### S1 — iOS Privacy Manifest

Apple requires a `PrivacyInfo.xcprivacy` file for all apps since Spring 2024. Missing or incomplete manifests are a common rejection reason.

**Check: File exists**
- Look for `ios/Runner/PrivacyInfo.xcprivacy` (or inside an `.xcframework`)
- Flag as Critical if absent

**Check: NSPrivacyTracking**
- Must be `true` or `false` — missing key defaults to `false`
- If the app uses any advertising/analytics SDKs, `true` requires an ATT prompt

**Check: NSPrivacyTrackingDomains**
- If `NSPrivacyTracking` is `true`, list all tracking domains
- Flag if `NSPrivacyTracking: true` but `NSPrivacyTrackingDomains` is empty

**Check: NSPrivacyCollectedDataTypes**
- Must list every category of data the app collects
- Common categories: `NSPrivacyCollectedDataTypeName`, `NSPrivacyCollectedDataTypeEmailAddress`, `NSPrivacyCollectedDataTypeDeviceID`, `NSPrivacyCollectedDataTypeCrashData`, `NSPrivacyCollectedDataTypePerformanceData`
- Cross-reference with the app's dependencies (Firebase, analytics SDKs) — their data collection must be declared

**Check: NSPrivacyAccessedAPITypes — Required Reason APIs**
- Apple requires a reason for using certain APIs. Check if the app or its dependencies use any of these and declare the reason:
  - `NSPrivacyAccessedAPICategoryFileTimestamp` — file timestamp APIs
  - `NSPrivacyAccessedAPICategorySystemBootTime` — system boot time
  - `NSPrivacyAccessedAPICategoryDiskSpace` — disk space APIs
  - `NSPrivacyAccessedAPICategoryActiveKeyboards` — keyboard list
  - `NSPrivacyAccessedAPICategoryUserDefaults` — UserDefaults
- See `references/store_checks.md` for the full list of required reason codes per API category

### S2 — Android Data Safety

Google Play requires a Data Safety section in the Play Console. This is declared in the console UI (not in the APK), but the app code must be consistent with what's declared.

**Check: Privacy policy URL**
- Verify a privacy policy URL is referenced in `AndroidManifest.xml` or the app's about screen
- Flag as High if no privacy policy URL exists anywhere in the project

**Check: Data collection consistency**
- Review what the app collects (auth, analytics, crash reporting, advertising IDs) and check that the codebase is consistent with what would need to be declared
- Flag: Firebase Analytics present → "App activity" data type must be declared
- Flag: Firebase Crashlytics present → "Crash logs" must be declared
- Flag: `android.permission.READ_CONTACTS` → "Contacts" must be declared
- Flag: `android.permission.ACCESS_FINE_LOCATION` → "Precise location" must be declared
- Flag: `android.permission.CAMERA` → "Photos and videos" may need to be declared if images are collected

**Check: Data safety form completeness signals**
- Look for any `<meta-data android:name="com.google.android.gms.ads.APPLICATION_ID"` → advertising data must be declared
- Check for any third-party analytics SDK in `pubspec.yaml` → usage data declaration needed

### S3 — Permissions

Both stores scrutinize permissions heavily. Unused or unjustified permissions cause rejections.

**iOS — Usage description strings**
Every permission key in `Info.plist` must have a human-readable purpose string. Check for:
- `NSCameraUsageDescription`
- `NSMicrophoneUsageDescription`
- `NSPhotoLibraryUsageDescription`
- `NSPhotoLibraryAddUsageDescription`
- `NSLocationWhenInUseUsageDescription`
- `NSLocationAlwaysAndWhenInUseUsageDescription`
- `NSContactsUsageDescription`
- `NSCalendarsUsageDescription`
- `NSBluetoothAlwaysUsageDescription`
- `NSFaceIDUsageDescription`
- `NSHealthShareUsageDescription` / `NSHealthUpdateUsageDescription`
- `NSSpeechRecognitionUsageDescription`

Flag: any usage description that is a placeholder (`"TODO"`, `"Required"`, `"App needs access"`) — Apple reviewers reject these
Flag: any usage description longer than ~100 words — keep them concise and specific

**Android — Manifest permissions**
- Read all `<uses-permission>` entries in `AndroidManifest.xml`
- Flag: `READ_CONTACTS`, `WRITE_CONTACTS`, `READ_CALL_LOG`, `RECORD_AUDIO`, `ACCESS_FINE_LOCATION`, `CAMERA` — these require explicit justification in the Data Safety form and may trigger manual review
- Flag: `MANAGE_EXTERNAL_STORAGE` — requires special Play Store approval
- Flag: `REQUEST_INSTALL_PACKAGES` — requires justification
- Flag: permissions declared but no corresponding feature or UI in the codebase → likely unused, should be removed
- Flag: `maxSdkVersion` not set for permissions that should be revoked after a certain SDK (e.g. `READ_EXTERNAL_STORAGE`)

### S4 — Screenshots & Store Assets

Missing or wrong-size assets cause rejection or prevent store listing.

See `references/store_checks.md` for the full size tables.

**iOS App Store — Required screenshots**
- iPhone 6.9" (1320×2868 or 1290×2796) — **required**, used as default
- iPhone 6.7" (1290×2796 or 1284×2778) — required for older devices
- iPad Pro 13" (2064×2752) — required if app supports iPad
- At least 1 screenshot per device class; maximum 10

**Google Play — Required assets**
- Feature graphic: 1024×500 px — **required** to be featured
- Phone screenshots: minimum 2, 16:9 or 9:16, between 320px and 3840px per side
- 7" tablet screenshots: required if app supports tablets
- 10" tablet screenshots: required if app supports tablets

**App icons**
- iOS: check `ios/Runner/Assets.xcassets/AppIcon.appiconset/Contents.json` — all required sizes present, no alpha channel (App Store rejects icons with transparency)
- Android: check `android/app/src/main/res/` for all mipmap densities: `mipmap-mdpi`, `mipmap-hdpi`, `mipmap-xhdpi`, `mipmap-xxhdpi`, `mipmap-xxxhdpi`
- Android: check for adaptive icon (`ic_launcher_foreground.xml` + `ic_launcher_background.xml`) for Android 8+

**What to flag**
- Missing iOS 6.9" screenshots → Critical (required since 2024)
- Missing feature graphic → High (app won't be featured, some placements require it)
- App icon with alpha channel on iOS → Critical (automatic rejection)
- Missing mipmap densities on Android → Medium

### S5 — Age Rating

Incorrect age rating causes rejection or app removal.

**iOS**
- Check `ios/Runner/Info.plist` for any content rating keys
- Check whether the app domain implies certain content descriptors:
  - Social features (user-generated content, messaging) → requires "Infrequent/Mild" or higher for mature themes
  - In-app purchases → must be declared
  - Location sharing → must be declared
  - Alcohol/tobacco references → must be declared
- Flag: health/fintech app with no age rating consideration → likely needs 4+ or 12+
- Flag: app with chat or user-generated content that claims 4+ age rating → likely incorrect

**Android**
- Check `android/app/src/main/AndroidManifest.xml` for `android:targetSandboxVersion`
- Check if `pubspec.yaml` includes any age-restricted SDKs (gambling, dating, alcohol)
- COPPA: if the app targets children (under 13), check:
  - No advertising SDKs unless COPPA-compliant
  - No analytics that collect device IDs
  - `android.permission.READ_PHONE_STATE` must be absent
  - Firebase Analytics must have `setAnalyticsCollectionEnabled(false)` for child users

**What to flag**
- Chat/UGC feature with a 4+ age rating claim → High
- Children's app with advertising SDKs → Critical
- No content rating consideration at all → Medium

### S6 — Metadata Length Limits

Text that exceeds store limits is truncated or rejected.

**iOS App Store limits** (check against any draft metadata in the project):
- App name: max **30 characters**
- Subtitle: max **30 characters**
- Description: max **4,000 characters**
- Keywords: max **100 characters** (comma-separated, no spaces after commas)
- What's New: max **4,000 characters**
- Promotional text: max **170 characters**

**Google Play limits**:
- App name: max **30 characters**
- Short description: max **80 characters**
- Full description: max **4,000 characters**
- What's new: max **500 characters**

**What to check in the codebase**
- Look for any `fastlane/` directory — check `Fastfile`, `metadata/` folder for stored metadata
- Look for `pubspec.yaml` `description` field — flag if it's being reused as a store description and is too long/short
- Look for any `store_listing/`, `assets/store/`, or `release/` folders with metadata files
- Check the app name in `pubspec.yaml` → `name:` and cross-reference with any hardcoded app name strings

**What to flag**
- App name in `pubspec.yaml` over 30 characters → High
- Fastlane metadata files with descriptions over limit → High
- No metadata files found at all → Info (manual check required before submission)

### S7 — iOS Specific Checks

**Info.plist**
- `CFBundleDisplayName` — should match the intended App Store name
- `CFBundleShortVersionString` — must be incremented from last submission
- `CFBundleVersion` — build number, must be unique per submission
- `LSApplicationQueriesSchemes` — any URL schemes queried must be justified
- `NSAppTransportSecurity` — flag any `NSAllowsArbitraryLoads: true` → High

**Export compliance**
- `ITSAppUsesNonExemptEncryption` in `Info.plist`:
  - `false` → declares no non-exempt encryption (fastest path)
  - `true` → requires export compliance documentation
  - Missing → App Store Connect will ask during submission

**Entitlements**
- Check `ios/Runner/Runner.entitlements`
- Flag: entitlements present that are not used in the app (e.g. `com.apple.developer.healthkit` with no HealthKit code)
- Flag: push notification entitlement without corresponding APNs configuration

**Minimum iOS version**
- Check `IPHONEOS_DEPLOYMENT_TARGET` in `ios/Runner.xcodeproj/project.pbxproj`
- Apple currently requires minimum iOS 16 for new submissions (verify against current Apple policy)

### S8 — Android Specific Checks

**build.gradle / build.gradle.kts**
- `targetSdkVersion` — Google Play requires targeting the latest or previous Android API level (currently 34+); flag if below
- `minSdkVersion` — flag if below 21 (Android 5.0) — Google Play's distribution floor has been raised
- `compileSdkVersion` — should match or exceed `targetSdkVersion`
- `versionCode` — must be incremented from last submission
- `versionName` — should match `pubspec.yaml` version

**Signing**
- Check `key.properties` exists (but should be gitignored)
- Check `build.gradle` has a `release` signing config referencing `key.properties`
- Flag: `build.gradle` using `debug` signing for the `release` build type → Critical
- Flag: `key.properties` committed to git → Critical (credential exposure)

**Bundle vs APK**
- Google Play now requires AAB (Android App Bundle) for new apps
- Check `build.gradle` for any config that forces APK output
- Flag: no AAB build configuration → High

**ProGuard / R8**
- Check `build.gradle` for `minifyEnabled true` in the release build type
- Check `proguard-rules.pro` exists and is not empty
- Flag: `minifyEnabled false` in release → Medium (larger bundle, no obfuscation)

**Manifest checks**
- `android:debuggable="true"` must not appear in release builds — flag if present in `AndroidManifest.xml`
- `android:allowBackup` — consider setting to `false` for security-sensitive apps
- `android:networkSecurityConfig` — check for any `cleartextTrafficPermitted="true"` → High

---

## Severity Guidelines

| Severity | Criteria |
|----------|----------|
| 🔴 Critical | Will cause automatic rejection: missing Privacy Manifest, debug signing on release, key.properties in git, alpha channel on iOS icon, children's app with non-COPPA ad SDK |
| 🟠 High | Likely rejection or major store policy violation: missing required screenshots, vague permission strings, no privacy policy, targetSdk below minimum |
| 🟡 Medium | Will delay or complicate submission: missing age rating consideration, no ProGuard in release, missing metadata files |
| 🔵 Low | Best practice gap: missing adaptive icon, metadata near length limit, build number not incremented |
| ℹ️ Info | Manual verification required — things that can only be confirmed in App Store Connect or Play Console |

---

## Important principles

- **Evidence only** — every finding needs a file path and the specific line or key.
- **Store policies change** — note when a finding is based on current policy and should be re-verified against official Apple/Google documentation before submission.
- **Separate iOS and Android** — clearly mark which findings apply to which platform.
- **Give credit** — note what is already correctly configured (privacy manifest present, signing configured, icons complete, etc.).

---

## Report Template

```
# App Store Submission Readiness Report

**App:** [name from pubspec.yaml]
**Date:** [today's date]
**Auditor:** Claude (AI-assisted static analysis)
**Scope:** iOS and Android submission readiness
**Flutter version target:** [from pubspec.yaml]

---

## Readiness Summary

| Platform | Status |
|----------|--------|
| iOS App Store | ✅ Ready / ⚠️ Issues found / ❌ Blocking issues |
| Google Play | ✅ Ready / ⚠️ Issues found / ❌ Blocking issues |

| Severity | Count |
|----------|-------|
| 🔴 Critical | N |
| 🟠 High | N |
| 🟡 Medium | N |
| 🔵 Low | N |
| ℹ️ Info | N |

[2–4 sentences: overall readiness, most critical blockers, and top recommendation.]

---

## iOS Checklist

### S1 — Privacy Manifest
| Item | Status | Notes |
|------|--------|-------|
| `PrivacyInfo.xcprivacy` exists | ✅ / ❌ | |
| `NSPrivacyTracking` declared | ✅ / ❌ | |
| `NSPrivacyCollectedDataTypes` complete | ✅ / ⚠️ / ❌ | |
| Required Reason APIs declared | ✅ / ⚠️ / ❌ | |

[Findings in standard format for any ❌ or ⚠️ items]

### S3 — Permissions (iOS)
| Permission | Usage Description | Status |
|-----------|-----------------|--------|
| Camera | "..." | ✅ / ❌ / ⚠️ |
| [etc.] | | |

### S4 — Screenshots & Assets (iOS)
| Asset | Required | Status |
|-------|---------|--------|
| iPhone 6.9" screenshots | Yes | ✅ / ❌ |
| iPad Pro 13" screenshots | If iPad supported | ✅ / ❌ / N/A |
| App icon (no alpha) | Yes | ✅ / ❌ |

### S5 — Age Rating (iOS)
[checklist]

### S6 — Metadata (iOS)
| Field | Limit | Current | Status |
|-------|-------|---------|--------|
| App name | 30 chars | N chars | ✅ / ❌ |
| [etc.] | | | |

### S7 — iOS Specific
| Item | Status | Notes |
|------|--------|-------|
| `ITSAppUsesNonExemptEncryption` set | ✅ / ❌ | |
| `NSAllowsArbitraryLoads` absent | ✅ / ❌ | |
| Minimum iOS version | ✅ / ⚠️ | |
| Entitlements match usage | ✅ / ⚠️ | |

---

## Android Checklist

### S2 — Data Safety
| Item | Status | Notes |
|------|--------|-------|
| Privacy policy URL present | ✅ / ❌ | |
| Data types consistent with code | ✅ / ⚠️ | |

### S3 — Permissions (Android)
| Permission | Justification | Status |
|-----------|--------------|--------|
| [permission] | [reason] | ✅ / ⚠️ |

### S4 — Screenshots & Assets (Android)
| Asset | Required | Status |
|-------|---------|--------|
| Feature graphic (1024×500) | Yes | ✅ / ❌ |
| Phone screenshots (min 2) | Yes | ✅ / ❌ |
| Adaptive icon | Recommended | ✅ / ❌ |

### S5 — Age Rating (Android)
[checklist]

### S6 — Metadata (Android)
| Field | Limit | Current | Status |
|-------|-------|---------|--------|
| App name | 30 chars | N chars | ✅ / ❌ |
| Short description | 80 chars | N chars | ✅ / ❌ |

### S8 — Android Specific
| Item | Status | Notes |
|------|--------|-------|
| `targetSdkVersion` ≥ 34 | ✅ / ❌ | |
| Release signing configured | ✅ / ❌ | |
| `key.properties` gitignored | ✅ / ❌ | |
| AAB build configured | ✅ / ❌ | |
| `minifyEnabled true` in release | ✅ / ❌ | |
| No `debuggable=true` in release | ✅ / ❌ | |

---

## Positive Findings

[What is already correctly configured.]

---

## Blocking Issues (Submit These First)

[Ordered list of Critical/High items that must be resolved before submission.]
```

---
> Source: [NoaTubic/flutter-store-readiness-skill](https://github.com/NoaTubic/flutter-store-readiness-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
