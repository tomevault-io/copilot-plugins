## nscbigwin

> This file provides guidance to coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Project

**Brivio** — AI-assisted adaptive learning mobile app for Thai high school students (NSC 2026 competition). Students sign up, select subjects/topics they've studied, take diagnostic assessments, and get personalized learning paths based on knowledge gaps.

**Constraints:** Flutter/Dart only, Android + Web, no backend (all local), setState for state management, weeks to deadline.

## Commands

```bash
# Run all tests
flutter test

# Run a single test file
flutter test test/auth_service_test.dart

# Run tests with coverage (requires lcov)
flutter test --coverage

# Analyze (lint)
flutter analyze

# Run on Chrome (web)
flutter run -d chrome

# Run on Android device/emulator
flutter run

# Format code
dart format lib/
```

## Architecture

### Layers
- **Presentation:** `lib/screens/` — StatefulWidget/StatelessWidget per screen, each in its own subdirectory. Private `_build*()` helpers below `build()`. Widgets for reuse in `widgets/` subdirectories.
- **Services:** `lib/services/` — Six service classes, all instantiated in `main.dart` and wired via top-level variables in `app_router.dart` (no DI container).
- **Models:** `lib/models/` — Immutable data classes with `toJson()`/`fromJson()`, `copyWith()` patterns.
- **Data:** `lib/data/` — 30 hardcoded `Question` objects across 3 subjects.
- **Router:** `lib/router/app_router.dart` — GoRouter with redirect guard.
- **Theme:** `lib/theme/app_theme.dart` — Material 3 dark theme via `ColorScheme.dark()`.

### Key Components

| Component | File | Notes |
|-----------|------|-------|
| BrivioApp | `lib/app.dart` | Root, wires theme + router |
| appRouter | `lib/router/app_router.dart` | 9 flat routes, redirect guard, extras for data passing |
| AuthService | `lib/services/auth_service.dart` | SHA-256 password hashing, salt, login/signup, ChangeNotifier for GoRouter refresh |
| UserService | `lib/services/user_service.dart` | CRUD for credentials via FlutterSecureStorage |
| ProfileService | `lib/services/profile_service.dart` | UserProfile persistence via Hive CE |
| SetupService | `lib/services/setup_service.dart` | Multi-step onboarding wizard, ChangeNotifier for GoRouter refresh, state save/restore |
| AssessmentService | `lib/services/assessment_service.dart` | Creates assessments, records responses, rule-based analysis |
| AssessmentHistoryService | `lib/services/assessment_history_service.dart` | Saves/loads AssessmentResult list via Hive CE |
| AppTheme | `lib/theme/app_theme.dart` | Dark theme, Inter font via GoogleFonts |

### Service Dependencies
```
AuthService ──> UserService (credentials)
SetupService ──> ProfileService (onboarding state)
AssessmentScreen ──> AssessmentService + AssessmentHistoryService
HomeScreen ──> ProfileService + AssessmentHistoryService + SetupService
```

### Persistence (Phase 03)
- **Hive CE** (`hive_ce` + `hive_ce_flutter`): `ProfileService` stores `UserProfile`, `AssessmentHistoryService` stores `List<AssessmentResult>`. Initialized in `main()` via `Hive.initFlutter()`.
- **FlutterSecureStorage** (`flutter_secure_storage`): `UserService` stores hashed credentials.
- **No backend** — everything is local on-device.

### Navigation & Data Flow

GoRouter with redirect guard (`app_router.dart:31-46`):
- Not logged in → redirect to `/login`
- Logged in but setup incomplete → redirect to `/onboarding`
- Logged in + setup complete → redirect to `/home` from auth/onboarding routes

Data passes between screens via `state.extra` as `Map<String, dynamic>` (type-unsafe — each screen manually casts keys with `as?` / `??` fallbacks).

Four main flows:
1. **Auth:** `/login` → `/signup` → `/onboarding` (or direct `/login` if returning user)
2. **Onboarding:** Multi-step wizard (grade → subjects → topics → summary), persisted via `SetupService`
3. **Assessment:** `/assessment` → questions → answers → `/analysis` → `/learning-path`
4. **Add subject:** `/home` → `/subjects` (isAddMode: true) → `/progress` (isAddMode: true) → `/assessment` → ... → pop back to `/home`

Route extras patterns:
- `SubjectSelectionScreen` receives `{'isAddMode': bool, 'enrolledSubjects': List<String>}`
- `ProgressSelectionScreen` receives either a `String` subject or `Map` with `subject` + `isAddMode`
- `AssessmentScreen` receives `{'subject': String, 'topics': List<String>}`
- `AnalysisScreen` / `LearningPathScreen` receive serialized `AnalysisResult` maps

## Conventions

- **Files:** snake_case.dart, one class per file, no barrel files
- **Classes:** PascalCase, private State with `_` prefix (`_LoginScreenState`)
- **Members:** camelCase, private with `_` prefix
- **Imports:** Relative paths only (`'../../theme/app_theme.dart'`), no `package:brivio/...`
- **Constructors:** Named parameters with `required`, `const` where possible
- **Models:** `toJson()` → snake_case keys, `fromJson()` as factory, `copyWith()` for clones
- **Services:** Stateless or ChangeNotifier, instantiated in `main()`, wired via top-level variables
- **Lint:** `flutter_lints` default config, `dart format` for formatting
- **Error handling:** 2 try/catch blocks exist (onboarding screen, home screen). Route extras use `as?` + `??` fallbacks. No error boundaries or error-state UI branches.
- **Logging:** None. `debugShowCheckedModeBanner: false`.

## Key Dependencies

| Package | Purpose |
|---------|---------|
| go_router ^14.0.0 | Declarative routing with redirect guard |
| google_fonts ^6.2.0 | Inter font |
| hive_ce ^2.19.3 + hive_ce_flutter ^2.3.4 | Local database (profiles, assessment history) |
| flutter_secure_storage ^10.2.0 | Encrypted credential storage |
| crypto ^3.0.7 | SHA-256 password hashing |
| uuid ^4.5.0 | Declared but not used in application code |
| cupertino_icons ^1.0.8 | Declared but unused (Material-only app) |
| shared_preferences ^2.5.5 | Declared but unused in application code |
| flutter_test (SDK) | Test framework |

## Testing

4 test files in `test/`:
- `widget_test.dart` — default smoke test
- `user_service_test.dart`
- `auth_service_test.dart`
- `assessment_history_service_test.dart`

Run with `flutter test`. Core analysis logic (`AssessmentService.analyze()`) has zero tests.

## Known Issues

- Route extras are type-unsafe `Map<String, dynamic>` — mismatched keys cause runtime null errors
- Hardcoded student ID `'ST001'` in assessment screen
- Duplicate assessment result saving (saved twice — in assessment screen and service)
- Subject display constants duplicated across 3 files
- "AI" label applied to rule-based if-else threshold logic (not ML)
- No input validation on login/signup screens
- No logout functionality
- 3 unused dependencies declared in pubspec.yaml

---
> Source: [Davie1423/NSCBIGWIN](https://github.com/Davie1423/NSCBIGWIN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
