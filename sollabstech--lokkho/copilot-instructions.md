## lokkho

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Lokkho** is a Flutter educational gamification app for Class 1-12 students in India. Students take MCQ tests, watch study videos, earn coins, and compete on leaderboards. Backend is Firebase (Firestore + Auth + Storage).

## Common Commands

```bash
# Install dependencies
flutter pub get

# Run app (debug)
flutter run

# Build Android APK (release)
flutter build apk --release

# Build iOS (release)
flutter build ios --release

# Analyze / lint
flutter analyze

# Format code
dart format lib/

# Run tests
flutter test

# Run single test file
flutter test test/widget_test.dart
```

## Architecture

### Pattern: GetX MVC with Feature Modules

Each feature lives under `lib/app/modules/<feature>/` and contains exactly three files:
- `*_binding.dart` — registers the controller via `Get.lazyPut()`
- `*_controller.dart` — extends `GetxController`; holds all business logic and `Rx<T>` state
- `*_view.dart` — stateless UI that reads state via `Obx()` or `GetBuilder<>()`

### Services (Permanent Singletons)

Registered in `main.dart` via `Get.put<T>(permanent: true)` before `runApp`:

| Service | Responsibility |
|---|---|
| `AuthService` | Firebase Auth + Google Sign-In; exposes `currentUser Rx<UserModel?>` |
| `FirestoreService` | All Firestore CRUD; includes mock data fallback for offline dev |
| `CoinService` | Coin economy — deduction on test start, rewards on login/video/completion |

Access anywhere via `Get.find<ServiceName>()`.

### Routing

- Route name constants: `lib/app/routes/app_routes.dart`
- Route → View + Binding mappings: `lib/app/routes/app_pages.dart`
- Navigate with `Get.toNamed(Routes.xyz)` or `Get.offAllNamed(Routes.splash)`

App flow: `/splash` → `/onboarding` (first run) or `/auth` → `/home` → feature screens

### State

All reactive state uses `.obs` / `Rx<T>`. UI rebuilds are wrapped in `Obx(() => ...)`. Avoid `StatefulWidget` unless animation controllers are needed.

### Data Models

All models in `lib/app/data/models/models.dart`. Every model has `fromMap(Map)` and `toMap()` for Firestore serialization. Key models: `UserModel`, `TestModel`, `QuestionModel`, `TestResultModel`, `CoinHistoryModel`, `LeaderboardModel`.

### Theme

Dark theme is the default (`lib/core/theme/dark_theme.dart`). Color palette: background `#0A0A1A`, primary `#6C63FF` (purple), secondary `#00D4FF` (cyan). Fonts: Poppins (headings) + Inter (body) via `google_fonts`. Glassmorphism cards via `glassDecoration()` helper in `dark_theme.dart`.

Constants (colors, strings, subject lists, size values) live in `lib/core/utils/constants.dart`.

## Key Configuration

### Firebase
- Android: `google-services.json` in `android/app/` (not committed — must be provided)
- Firebase BOM `34.6.0` declared in `android/app/build.gradle.kts`
- Firestore security rules: `firestore.rules` — users read/write only their own doc; tests/leaderboard/study materials require auth

### Android Release Signing
Release builds require a `gradle.properties` (not committed) with:
```
RELEASE_STORE_FILE=<path-to-keystore>
RELEASE_STORE_PASSWORD=...
RELEASE_KEY_ALIAS=...
RELEASE_KEY_PASSWORD=...
```

### System UI (main.dart)
- Locked to portrait orientation
- Transparent status bar, dark navigation bar (`#0A0A1A`)
- Default page transition: 300ms `fadeIn`

## Coin Economy Logic

Tests cost coins to start (deducted by `CoinService`). Rewards: +5 daily login, +2 per video watched, variable completion bonus based on accuracy. All coin events are logged as `CoinHistoryModel` documents in Firestore under the user's sub-collection.

---

## Machine-Specific Build Notes (Windows Dev Machine)

### Connected Device
- **Motorola Edge 60 Pro** — device ID `ZA223977L7`, Android 16 (API 36), arm64
- Connect via USB, enable USB debugging, run `adb devices` to confirm before `flutter run -d ZA223977L7`

### Known Gradle Build Error — "Unable to establish loopback connection"
**Root cause:** Java's `WEPollSelectorImpl` (used internally by Gradle's daemon protocol) fails on this machine because its Unix domain socket pipe implementation is broken on Windows 11 Build 26200 (Insider Dev channel). This affects both JBR 21 (Android Studio) and Temurin JDK 17.

**Affected files already patched:**
- `android/gradle.properties` — has `org.gradle.daemon=false` and custom `jvmargs` with `-Djava.io.tmpdir=C:/gradle_home/tmp`
- `android/gradlew.bat` — `DEFAULT_JVM_OPTS` set to `-Djava.io.tmpdir=C:\gradle_home\tmp`
- `android/gradlew` — `DEFAULT_JVM_OPTS` set with same tmpdir

**Workaround directory:** `C:\gradle_home\` — created to avoid the 8.3 short-path issue (`DRRADI~1`) with Unix domain sockets on Windows.
- `C:\gradle_home\tmp\` — clean tmpdir for Java (no spaces, no 8.3 short names)
- `C:\gradle_home\jdk17\jdk-17.0.12+7\` — Adoptium Temurin JDK 17 (downloaded)

**Status as of 2026-06-14:** The loopback error is NOT yet fully resolved. The daemon JVM gets the correct tmpdir but the Gradle wrapper's selector still fails. Options still to try:
1. `flutter config --jdk-dir "C:\gradle_home\jdk17\jdk-17.0.12+7"` then run again (JDK 11 would be better — JDK 11 predates WEPoll entirely)
2. Run/build directly from **Android Studio** (its own Gradle runner may bypass the issue)
3. Downgrade Windows Insider build or switch back to stable Windows channel

### JDK Path
- Android Studio JBR: `C:\Program Files\Android\Android Studio\jbr\`
- Temurin JDK 17 (downloaded): `C:\gradle_home\jdk17\jdk-17.0.12+7\`
- Configure Flutter to use a specific JDK: `flutter config --jdk-dir "C:\path\to\jdk"`

### Android SDK
- Located at `C:\Android\sdk`
- `ANDROID_HOME` and `ANDROID_SDK_ROOT` both point there

---
> Source: [sollabstech/lokkho](https://github.com/sollabstech/lokkho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
