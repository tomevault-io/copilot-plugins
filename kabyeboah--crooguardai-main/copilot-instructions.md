## crooguardai-main

> - Install deps: `flutter pub get`

# Copilot instructions for CropGuard AI

## Build, test, and lint

- Install deps: `flutter pub get`
- Run app: `flutter run`
- Lint/analyze: `flutter analyze`
- Run all tests: `flutter test`
- Run one test file: `flutter test test/domain/usecases/scanner/scan_crop_usecase_test.dart`
- Another single test file: `flutter test test/domain/usecases/auth/login_usecase_test.dart`

## High-level architecture

- This is a Flutter app built with Clean Architecture.
- `lib/main.dart` bootstraps Firebase, GetIt, notifications, background tasks, and app startup before `runApp`.
- `lib/app.dart` wires `MaterialApp.router`, theming, and localization.
- `lib/presentation/navigation/app_router.dart` owns all routing, including the `ShellRoute` for the bottom-nav shell.
- `lib/core/di/service_locator.dart` registers data sources, repositories, use cases, and top-level providers.
- Flow is `presentation -> domain -> data`; UI should not talk directly to Firebase, SQLite, or TFLite.

## Key conventions

- Keep business logic in use cases and repository implementations, not in widgets.
- Use `Result<T>` and `Failure` for fallible operations; map failures in providers to UI state.
- Providers are `ChangeNotifier`-based and live under `lib/presentation/screens/*/*_provider.dart`.
- Use shared components and theme tokens from `lib/presentation/components` and `lib/core/theme` instead of one-off UI.
- Localization is generated from `l10n/app_*.arb`; use `AppLocalizations` and keep user-facing copy out of widgets when a resource exists.
- Supported locales are `en`, `tw`, `ee`, and `dag`.
- The scan pipeline is coupled to the current ML contract: `CropDiseaseClassifier` uses a bundled TFLite model, and `ScanCropUseCase` maps confidence to severity. Keep model assets, labels, and thresholds in sync.
- Tests use `flutter_test` and `mocktail`; follow the existing unit-test style in `test/domain/usecases/*`.
- Assets live under `assets/`; update `pubspec.yaml` when adding model files, labels, or API specs.

---
> Source: [kabyeboah/CrooGuardAI-MAIN](https://github.com/kabyeboah/CrooGuardAI-MAIN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
