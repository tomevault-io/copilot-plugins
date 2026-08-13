## palmx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

PalmX is a Flutter app for logging and costing palm-plantation field operations (activities like spraying, harvesting, pruning), tracked per field/date with detailed cost breakdowns (labour, supervision, driver, material, evit). All data is local-only — there is no backend; a `dio`-based HTTP client exists but is not currently wired to real endpoints.

## Commands

```bash
flutter pub get                          # install dependencies
dart run build_runner build --delete-conflicting-outputs   # regenerate code (drift tables, injectable DI)
# Faster, scoped regen after editing DI only:
dart run build_runner build --delete-conflicting-outputs --build-filter="lib/core/di/injection.dart"

flutter analyze                          # static analysis / lint
flutter test                             # run all tests
flutter test test/widget_test.dart       # run a single test file

flutter run                              # run in debug mode
scripts/build.sh                         # interactive release/debug build for Android/iOS, stamps build number from pubspec.yaml
scripts/bump_patch.sh                    # bump patch + build number in pubspec.yaml (also bump_minor.sh, bump_major.sh)
```

Code generation must be re-run after changing:
- Any Drift table in `lib/data/local/tables/` or the `@DriftDatabase` annotation in `lib/core/local/database.dart` → regenerates `database.g.dart`.
- Any `@injectable`/`@lazySingleton`/`@module`-annotated class → regenerates `lib/core/di/injection.config.dart`.

## Architecture

**Layering (per feature, under `lib/features/<feature>/`):** `presentation/` (pages + `ChangeNotifier` providers) → `domain/` (usecases + repository abstract interfaces) → `data/` (repository impls + datasources). Not every feature has all three layers yet — `home` only has `domain/usecase` + `presentations` (queries go straight to a shared datasource); `operation` is the fullest example of the complete pattern. Follow the `operation` feature as the template when adding a new one.

- Repositories return `Either<Failure, T>` from `fpdart` for fallible operations (`lib/core/error/failure.dart` is a single-field message wrapper), and raw `Stream<T>` for reactive queries (no Either wrapping on streams).
- Datasources talk directly to the Drift `AppDatabase`; repository impls catch exceptions and convert them to `Failure`.

**Dependency injection:** `get_it` + `injectable`. `lib/core/di/injection.dart` calls the generated `injection.config.dart`; annotate classes with `@injectable`, `@lazySingleton`, or `@LazySingleton(as: SomeInterface)` (used to bind repository impls to their abstract interface) and provide manual bindings via `@module` classes like `lib/core/local/di/database_module.dart`. Access instances via `sl<T>()` (alias for `getIt<T>()`) defined in `lib/core/provider/app_provider.dart`.

**State management:** `provider` package, one `ChangeNotifierProvider` per feature, all registered in `AppProviders.providers` (`lib/core/provider/app_provider.dart`) and provided app-wide via `MultiProvider` in `main.dart`. Providers are resolved through DI (`sl<XProvider>()`), not constructed inline. Cross-provider communication happens via `context.read<OtherProvider>()` (e.g. `OperationProvider.submit()` pushes into `CalendarProvider` after a successful save).

**Local persistence:** `drift` (SQLite via `sqflite`/`NativeDatabase`) is the single source of truth. `AppDatabase` (`lib/core/local/database.dart`) is a private singleton (`factory AppDatabase()`) storing `palmx.db` in the app documents directory. Tables live in `lib/data/local/tables/`, re-exported via `tables.dart`. Migrations/seeding go through `DatabaseMigration.strategy()` in `lib/core/local/migrations/schema_versions.dart` — `onCreate` seeds initial `ActivityTable`/`MaterialTable` reference data (activity cost types, material prices) from the hardcoded `activityUnitMap`/`materialCostMap`. `DatabaseBackupService` (`lib/data/local/services/`) exports/imports the whole DB as a shared JSON file via `share_plus`/`file_picker`.

**Navigation:** No named routes/router package. `MainPage` (`lib/features/main-navigation/main_page.dart`) hosts a `PageView` with a custom floating bottom nav bar for the three tabs (Home, Calendar, Settings) plus a FAB that pushes `OperationLogFormPage` via plain `Navigator.push`. `AppNavigator.navigatorKey` (`lib/core/service/app_navigator.dart`) is set as `MaterialApp.navigatorKey` so navigation/dialogs can be triggered outside widget context (e.g. from services).

**Notifications:** `NotificationService` (`lib/core/notifications/`) wraps `flutter_local_notifications`, is a no-op on web/Linux, and supports one-off, periodic, and scheduled (`zonedSchedule`, timezone-aware) notifications. Foreground/background tap handlers are split into `notification_foreground.dart`/`notification_background.dart`.

**Networking:** `DioClient` (`lib/core/network/dio_client/`) is a singleton wrapping `dio` with `AuthInterceptor`, `LogInterceptor`, and `RetryInterceptor`. Not currently consumed by any feature — treat as scaffolding for future remote sync, not active infrastructure.

**App startup:** `main.dart` → `AppInitializer.init()` (`lib/core/app_initializer.dart`) runs DI configuration, notification init, EasyLoading config, and forces portrait orientation, all before `runApp`.

**Models vs. Drift row types:** `lib/data/local/models/` defines plain Dart model classes (e.g. `OperationLogModel`) distinct from the Drift-generated `*TableData`/`*TableCompanion` types — models are used in providers/UI and converted to Companions via `toInsert()` before hitting the repository layer.

## Naming conventions to be aware of

- Feature directories mix `presentation` and `presentations` (singular in `operation`/`settings`, plural in `home`) — check the actual folder before assuming.
- `lib/features/settings/dropdown-form/` uses kebab-case unlike the rest of `lib/`, which is snake_case.

---
> Source: [Muitsu/PalmX](https://github.com/Muitsu/PalmX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
