## hesti-surgery

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hesti Surgery is a 3D surgical planning application for rhinoplasty, built with Flutter. It uses Supabase as the backend (auth, database, storage) and FAL AI for 3D model generation. Supports English and Turkish localization.

## Build & Development Commands

```bash
# Install dependencies
flutter pub get

# Run code generation (auto_route, freezed, json_serializable, hive adapters)
dart run build_runner build --delete-conflicting-outputs

# Run the app
flutter run

# Run all tests
flutter test

# Run a single test file
flutter test test/features/auth/domain/usecases/login_user_test.dart

# Analyze code
flutter analyze

# Format code
dart format .
```

## Environment Setup

Copy `.env.example` to `.env` and fill in:
- `SUPABASE_URL` - Supabase project URL
- `SUPABASE_ANON_KEY` - Supabase anonymous key
- `FAL_AI_API_KEY` - FAL AI API key

## Architecture

Clean Architecture with BLoC state management, organized by feature modules.

### Layer Flow

```
UI (Pages/Widgets) → BLoC/Cubit → UseCase → Repository (abstract) → DataSource
                                                    ↓
                                            RepositoryImpl → Remote (Supabase) / Local (Hive/SharedPreferences)
```

### Error Handling Convention

Two-tier system: DataSources throw **Exceptions** (`ServerException`, `CacheException`, `NetworkException`, `ParseException`), repositories catch them and return **Failures** (`ServerFailure`, `CacheFailure`, `NetworkFailure`, `ValidationFailure`) via the `Either<Failure, T>` pattern from dartz.

### UseCase Pattern

All use cases extend `UseCase<T, Params>` returning `Future<Either<Failure, T>>`. Use `NoParams` when no input is needed.

### Dependency Injection

GetIt service locator in `lib/injection_container.dart`. Global instance is `sl`. Pattern:
- `registerFactory` for BLoCs/Cubits (new instance per request)
- `registerLazySingleton` for everything else (repositories, data sources, use cases, core services)

Each feature has a private `_initXFeature()` method that registers: BLoC → UseCases → Repository → DataSources.

### Offline-First

`OfflineManager` orchestrates Hive local storage, `ConnectivityService` for network monitoring, and `SyncQueue` for pending operations. Auto-syncs when connectivity returns. `ConnectivityCubit` provides UI integration.

## Feature Module Structure

Each feature under `lib/features/` follows:
```
feature_name/
├── domain/
│   ├── entities/        # Pure Dart entities (extend Equatable)
│   ├── repositories/    # Abstract repository interfaces
│   └── usecases/        # Business logic (one class per use case)
├── data/
│   ├── models/          # Extend entities, add JSON serialization
│   ├── datasources/     # Abstract + implementations (remote/local)
│   └── repositories/    # Implement domain repositories
└── presentation/
    ├── bloc/            # BLoC (events/states) or Cubit
    ├── pages/           # Full screens
    └── widgets/         # Feature-specific UI components
```

**Implemented features:** auth, patient, surgery_case, photo_capture, model_3d, measurements, dashboard, settings, home, main_navigation, report.

## Routing

Auto_route with code generation. Router defined in `lib/config/routes/app_router.dart`, generates `app_router.gr.dart`. Route flow: Login → MainNavigation (Dashboard, PatientList, Settings tabs).

After modifying routes, run `dart run build_runner build --delete-conflicting-outputs`.

## State Management

- **BLoC** for complex feature state (AuthBloc, PatientListBloc, SurgeryCaseBloc)
- **Cubit** for simple state (ThemeCubit, LocaleCubit, ConnectivityCubit)
- BLoCs are provided at the app level in `lib/app.dart`

## Database Schema (Supabase)

Migrations in `supabase/migrations/`. Key tables: `surgeon_profiles` (extends auth.users), `patients`, `surgery_cases`, `photos`, `models_3d`, `measurements`. All have RLS policies scoped to the owning surgeon. Storage buckets: `patient-photos` (10MB, JPEG/PNG), `3d-models` (50MB, GLB/OBJ), `reports` (20MB, PDF).

## Testing

- Uses `mocktail` for mocking (not mockito)
- Centralized mocks in `test/mocks/mocks.dart` — add new mocks there
- Test helpers in `test/helpers/test_helpers.dart` (TestData factory)
- JSON fixtures in `test/fixtures/`
- BLoC tests use `bloc_test` package with `blocTest()` helper
- Register fallback values via `registerFallbackValues()` in test setup

## Code Generation

Generated files (`*.g.dart`, `*.freezed.dart`, `*.gr.dart`) are excluded from analysis. Always regenerate after modifying:
- Route definitions → `app_router.gr.dart`
- Freezed/JSON-annotated models → `*.freezed.dart`, `*.g.dart`
- Hive type adapters → `*.g.dart`

## Naming Conventions

- Files: `snake_case.dart`
- Classes: `PascalCase` with suffixes — `Entity`, `Model`, `Repository`, `RepositoryImpl`, `Bloc`, `Cubit`, `Event`, `State`, `Page`, `Widget`, `DataSource`, `UseCase`
- Test files: `*_test.dart` mirroring source path
- Test data: `t` prefix (`tUser`, `tParams`), mocks: `Mock` prefix

## Analysis Rules

Strict mode enabled (`strict-casts`, `strict-inference`, `strict-raw-types`). Uses `flutter_lints` with extensive custom rules. Enforces `prefer_const_constructors`, `avoid_print`, `unawaited_futures`, `always_declare_return_types`, among others. See `analysis_options.yaml` for full list.

---
> Source: [cihatapp/hesti_surgery](https://github.com/cihatapp/hesti_surgery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
