## todoflutterapp

> A cross-platform Flutter to-do app with Firebase sync, local SQLite storage (Drift), and Riverpod state management. Targets Android and iOS.

# TodoFlutterApp — Copilot Instructions

A cross-platform Flutter to-do app with Firebase sync, local SQLite storage (Drift), and Riverpod state management. Targets Android and iOS.

## Commands

```sh
flutter pub get                           # Install dependencies
flutter run                               # Run on connected device/emulator
flutter run -d chrome                     # Run as web app
flutter build apk                         # Android APK
flutter build appbundle                   # Android App Bundle (Play Store)
flutter test                              # All unit & widget tests
flutter test test/features/auth/          # Tests for a specific feature
flutter analyze                           # Dart static analysis
dart fix --apply                          # Auto-fix lint issues
```

## Architecture

Feature-based folder structure under `lib/`:

```
lib/
  main.dart                  App entry point; initialises Firebase, Drift, Riverpod
  app/
    app.dart                 MaterialApp root with go_router and theme
    router.dart              All route definitions (go_router)
    theme.dart               App colour scheme and TextTheme
  features/
    auth/                    Email/Google sign-in, auth state stream
    home/                    Task list screen and shimmer loading state
    task/                    Create/edit task form, task detail screen
    profile/                 User profile, settings
  shared/
    models/                  Drift table definitions and generated query classes
    providers/               Riverpod providers (global singletons)
    widgets/                 Reusable UI widgets (TaskCard, LoadingOverlay, etc.)
    utils/                   Date formatting, validators, extensions
  data/
    local/                   Drift database class, DAOs
    remote/                  Firestore repository classes
test/
  features/                  Unit tests, mirroring lib/features/ structure
  shared/                    Widget tests for shared components
```

## Key Conventions

- **Riverpod everywhere.** State is managed via `ref.watch`/`ref.read` on providers. Do not use `StatefulWidget` for business logic — use `ConsumerWidget`/`ConsumerStatefulWidget`.
- **Drift for local data.** Table schemas live in `lib/shared/models/`. After changing a schema, run `flutter pub run build_runner build --delete-conflicting-outputs` to regenerate.
- **Firebase Auth + Firestore for cloud.** Firestore rules enforce per-user document access — match `userId` fields when writing queries.
- **go_router for navigation.** Define new screens in `app/router.dart` as `GoRoute` entries. Use `context.go()` / `context.push()` for navigation; never use `Navigator.push` directly.
- **Test every feature folder.** Add tests under `test/features/<feature>/` for new providers and use cases.

---
> Source: [mihassan/TodoFlutterApp](https://github.com/mihassan/TodoFlutterApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
