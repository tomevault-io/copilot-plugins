## willup

> Application code lives in `lib/`: `app` for routing/theming, `core` for cross-cutting services, domain flows in `features/<name>`, and shared UI in `shared`. Alarm audio and other resources reside in `assets/audio/`; register additions in `pubspec.yaml`. Platform shells stay in `android/`, `ios/`, `macos/`, `windows/`, `linux/`, and `web/`. Product notes belong in `specs/`, and new tests should mirror feature folders under `test/`.


# Repository Guidelines

## Project Structure & Module Organization
Application code lives in `lib/`: `app` for routing/theming, `core` for cross-cutting services, domain flows in `features/<name>`, and shared UI in `shared`. Alarm audio and other resources reside in `assets/audio/`; register additions in `pubspec.yaml`. Platform shells stay in `android/`, `ios/`, `macos/`, `windows/`, `linux/`, and `web/`. Product notes belong in `specs/`, and new tests should mirror feature folders under `test/`.

## Build, Test, and Development Commands
- `flutter pub get` installs dependencies after cloning or updating `pubspec.yaml`.
- `flutter analyze` applies the lint rules from `analysis_options.yaml`.
- `dart run build_runner build --delete-conflicting-outputs` regenerates Freezed, Riverpod, and JSON outputs.
- `flutter test` runs automated tests; add `--coverage` when validating larger changes.
- `flutter run -d <device_id>` launches the app on a connected simulator or device.

## Coding Style & Naming Conventions
Follow Flutter's two-space indentation and the `flutter_lints` ruleset; address analyzer findings before opening a pull request. Use `snake_case.dart` for files, `UpperCamelCase` for classes and enums, and `lowerCamelCase` for variables and functions; suffix Riverpod providers with `Provider`. Format with `dart format lib test`, favor `const` constructors when feasible, and centralize reusable UI in `lib/shared`.

## Testing Guidelines
Place new suites in `test/<feature>`, keep filenames ending in `_test.dart`, and group cases with descriptive `group` labels. Target provider or widget scenarios that cover scheduling, notifications, and navigation flows. Run `build_runner` beforehand so generated stubs compile, and document any manual platform checks in the pull request.

## Commit & Pull Request Guidelines
Match the repo’s Conventional Commit style (`feat:`, `fix:`, `chore:`), keep summaries imperative, and stay near 72 characters. Squash or rebase before opening a pull request and include: a concise summary, linked issue or relevant spec entry, screenshots for UI changes, and a list of local tests executed. Flag migrations, new environment variables, or manual deployment tasks so reviewers can plan verification.

## Environment & Configuration Tips
Secrets load through `flutter_dotenv`; request the current `.env` template from the team and never commit real credentials. Clear caches such as Hive boxes with `flutter clean` if local data conflicts arise. When adding audio or mission assets, keep sizes small and verify alarm flows on simulator and physical hardware.

---
> Source: [zyeh0001/willup](https://github.com/zyeh0001/willup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
