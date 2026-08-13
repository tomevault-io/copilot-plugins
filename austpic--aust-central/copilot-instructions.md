## aust-central

> Notes for agents working in this repo. Verify against the code before relying on anything below.

# AGENTS.md

Notes for agents working in this repo. Verify against the code before relying on anything below.

## Project

`aust_track` — a Flutter app (AUST Central / AUSTrack) for Ahsanullah University students: transport, QR, lost-and-found, notes. Firebase email/password auth.

The repo directory is `aust-central`, but the Dart package (and every import) is `aust_track`. Always use `package:aust_track/...` in imports.

## Commands

Flutter SDK lives at `/home/frostflux/development/flutter` (`flutter`, `dart` on PATH).

```bash
flutter pub get          # after pulling dependency changes
flutter run              # runs the currently connected device/emulator
flutter analyze          # lint/typecheck — run before considering work done
flutter test             # see "Tests" caveat below
flutter build apk        # Android release/debug APK
```

Lint config: `analysis_options.yaml` extends `package:flutter_lints/flutter.yaml`.

## Firebase / platform constraints (important)

`lib/main.dart` calls `Firebase.initializeApp()` with **no Dart options** — there is no `firebase_options.dart` / `DefaultFirebaseOptions`. The app therefore depends on native platform config:

- Android: `android/app/google-services.json` is present (Firebase project `austrack-d8397`). **Android is the only platform that boots end-to-end.**
- iOS / macOS: `GoogleService-Info.plist` is **missing**.
- Web: no injected web config.

Launching on iOS/web/desktop will crash at `initializeApp()`. Don't try to "fix" this by adding `firebase_options.dart` unless asked — instead surface it.

## App entry / navigation flow

`main.dart` → Firebase init → `SplashScreen` (3s timer) → `WelcomeScreen` → `LoginScreen`/`RegisterPage` → `Navigation` (`bottomnavigation_page.dart`), a `GNav` bottom bar with 5 tabs in this order: **Home, Bus (Transport), MissingNotes, QR, LostFound**. Each tab is a top-level file in `lib/`.

Auth logic is centralized in `lib/services/auth_service.dart` (`AuthService` wraps `FirebaseAuth`).

## Code layout

Flat `lib/` — every screen is a top-level `.dart` file, not grouped by feature. The only subfolder is `lib/services/`. Keep this structure when adding screens.

Brand color used throughout: `Color(0xff407362)`.

## Tests (caveat)

`test/widget_test.dart` is the **untouched Flutter counter template** — it expects a counter (`find.text('0')`, `Icons.add`) that does not exist in this app, and it pumps `MyApp()` without Firebase init. **`flutter test` currently fails.** Don't treat red tests as a regression you caused, and don't write new tests assuming the existing one is a working pattern.

## Android build notes

- `applicationId` / namespace is still `com.example.aust_track` (placeholder — not been renamed).
- `compileSdk`/`targetSdk` = 36, NDK 28, Java/Kotlin 17.
- Release builds sign with the **debug** keystore (`android/app/build.gradle.kts`) — fine for local APKs, not for Play Store.

---
> Source: [austpic/aust-central](https://github.com/austpic/aust-central) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
