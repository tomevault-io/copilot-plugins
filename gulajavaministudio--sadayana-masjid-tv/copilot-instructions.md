## sadayana-masjid-tv

> Aplikasi digital signage berbasis Flutter untuk **Android TV** (16:9 Landscape, 1920×1080).

# Miqotul Khoir TV — Gemini Context

Aplikasi digital signage berbasis Flutter untuk **Android TV** (16:9 Landscape, 1920×1080).
Untuk panduan lengkap, lihat `AGENTS.md`.

## Struktur Proyek

- `lib/core/` — Utilities, constants, service layer
  - `lib/core/theme/islamic_colors.dart` — 21 color constants, Islamic Glassmorphism (DONE)
  - `lib/core/theme/islamic_typography.dart` — 7 text styles, Poppins + .sp (DONE)
  - `lib/core/theme/islamic_theme.dart` — Material3 dark ThemeData (DONE)
  - `lib/core/theme/tv_safe_area.dart` — 5% margin TV safe area wrapper (DONE)
- `lib/data/` — SQLite models, repositories, data sources
  - `lib/data/datasources/database_helper.dart` — Singleton DatabaseHelper (DONE)
  - `lib/data/datasources/settings_local_data_source.dart` — Settings SQLite ops (DONE)
  - `lib/data/datasources/city_local_data_source.dart` — City SQLite ops (DONE)
  - `lib/data/models/settings_model.dart` — fromMap/toMap (DONE)
  - `lib/data/models/city_model.dart` — fromMap/toMap (DONE)
  - `lib/data/repositories/settings_repository_impl.dart` — SHA-256 PIN, delegation (DONE)
  - `lib/data/repositories/city_repository_impl.dart` — delegation (DONE)
- `lib/domain/` — Business logic, entities, use cases
  - `lib/domain/entities/settings.dart` — Immutable Settings entity, 27 fields (DONE)
  - `lib/domain/entities/city.dart` — Immutable City entity (DONE)
  - `lib/domain/repositories/settings_repository.dart` — Abstract interface (DONE)
  - `lib/domain/repositories/city_repository.dart` — Abstract interface (DONE)
  - `lib/domain/usecases/` — Application business rules
    - `calculate_prayer_times_use_case.dart` — Prayer time calculation logic (DONE)
    - `evaluate_display_state_use_case.dart` — Display state evaluation logic (DONE)
- `lib/presentation/` — UI, pages, Cubit state management
  - `lib/presentation/cubits/` — State management
    - `prayer_time/` — Prayer time calculation state (DONE)
    - `display_state/` — Display state machine (DONE)
    - `setup_wizard/` — Setup wizard logic (DONE)
    - `settings/` — Settings logic, auto-save, PIN (DONE)
  - `lib/presentation/widgets/glassmorphism_card.dart` — BackdropFilter blur card, isFocused golden glow (DONE)
  - `lib/presentation/widgets/focusable_widget.dart` — D-Pad focus wrapper, AnimatedScale, KeyEvent handler (DONE)
  - `lib/presentation/widgets/islamic_background.dart` — Layered gradient, anti burn-in Timer (DONE)
  - `lib/presentation/widgets/running_text_widget.dart` — Marquee horizontal scroll, empty guard (DONE)
  - `lib/presentation/widgets/step_indicator_widget.dart` — Setup wizard visual progress (DONE)
  - `lib/presentation/pages/setup_wizard/` — Setup Wizard feature (DONE)
    - `setup_wizard_page.dart` — Root navigation & provider
    - `steps/welcome_step.dart` 
    - `steps/identity_step.dart`
    - `steps/location_step.dart`
    - `steps/preview_step.dart`
  - `lib/presentation/pages/splash_page.dart` — Startup routing logic (DONE)
- `test/` — Unit, widget, dan integration tests
  - `test/data/datasources/database_helper_test.dart` — 6 unit tests (DONE)
  - `test/data/models/settings_model_test.dart` — 4 unit tests (DONE)
  - `test/data/models/city_model_test.dart` — 3 unit tests (DONE)
  - `test/data/repositories/settings_repository_impl_test.dart` — 5 unit tests (DONE)
  - `test/data/repositories/city_repository_impl_test.dart` — 4 unit tests (DONE)
  - `test/core/theme/islamic_colors_test.dart` — 15 unit tests (DONE)
  - `test/core/theme/islamic_typography_test.dart` — 10 unit tests (DONE)
  - `test/core/theme/islamic_theme_test.dart` — 17 unit tests (DONE)
  - `test/presentation/widgets/glassmorphism_card_test.dart` — 5 widget tests (DONE)
  - `test/presentation/widgets/focusable_widget_test.dart` — 7 widget tests (DONE)
  - `test/presentation/widgets/running_text_widget_test.dart` — 6 widget tests (DONE)
  - `test/presentation/pages/setup_wizard/setup_wizard_page_test.dart` — Widget tests for all steps (DONE)
  - `test/domain/usecases/` — Use case tests (DONE)
  - `test/presentation/cubits/` — Cubit tests (DONE)
- `assets/data/` — JSON seed data (cities.json: 514 kota, 34 provinsi)
- `spec/` — Technical specification (output @SpecificationArchitect)
- `plan/` — Implementation plans dengan task tracking
- `docs/` — Design documents dan technical guides
- `tools/` — Helper scripts (generate_cities.py)

## Konvensi Teknis Wajib

- **State Management**: Cubit (bukan Bloc events). Semua feature pakai Cubit.
- **Data Persistence**: SQLite offline-first, single source of truth
- **Responsive UI**: `flutter_screenutil` — design baseline 1920×1080
  - Font → `.sp`, width → `.w`, height → `.h`, radius → `.r`
  - Hardcoded pixel values **dilarang** — selalu gunakan ScreenUtil extensions
- **Navigation**: D-Pad first (remote control), semua elemen harus focusable
- **Theme**: Islamic Glassmorphism — Deep Emerald Green + Gold/Amber accents
- **Prayer Time**: `adhan-dart` + manual correction
- **Testing**: `flutter test --reporter=expanded` (wajib format ini)

## Workflow SDLC (Urutan Wajib)

PRD → Spec → Plan → Code → Test → Docs

- **Tidak boleh melompat fase** — setiap tahap harus selesai sebelum lanjut
- **Session baru per fase** — untuk menjaga fokus konteks
- **Setelah fitur selesai** → update `.github/instructions/memory.instructions.md`
- **Git Control**: **DILARANG** melakukan commit atau push otomatis. Selalu minta izin user sebelum commit/push.

## Komunikasi

- **Bahasa**: Indonesia, formal tapi santai
- **Penjelasan**: Jelas, terstruktur, detail teknis yang komprehensif
- **Dokumentasi**: Lengkap, mudah dipahami, lolos markdown lint
- **Format**: Heading, bullet points, code blocks, tabel sesuai kebutuhan

## Referensi Lengkap

- `AGENTS.md` — Panduan kontributor lengkap + patterns + examples
- `docs/SPECIFICATION_OVERVIEW.md` — Overview 6 spec documents + dependency map + design decisions
- `docs/ARCHITECTURE_PATTERNS.md` — State machine, offline-first, timer patterns
- `docs/UI_UX_GUIDE.md` — Android TV design, ScreenUtil, glassmorphism
- `docs/DEVELOPMENT_WORKFLOW.md` — Git workflow, testing, code review
- `docs/TESTING_GUIDE.md` — Testing strategies (SQLite, Cubit, widget)
- `docs/EXECUTION_WORKFLOW.md` — Phased execution + checkpoint gates

## Implementation Status

| Plan | Scope | Status |
|------|-------|--------|
| Plan 01 (`infrastructure-database-1.md`) | DatabaseHelper, schema DDL, seed data, migration, tests | ✅ COMPLETED |
| Plan 02 (`feature-data-layer-1.md`) | Entities, models, repository interfaces & implementations, 22 tests | ✅ COMPLETED |
| Plan 03 (`design-theme-system-1.md`) | Colors, typography, ThemeData, ScreenUtil, TV safe area, 42 tests | ✅ COMPLETED |
| Plan 04 (`design-ui-components-1.md`) | GlassmorphismCard, FocusableWidget, IslamicBackground, RunningTextWidget, 20 tests | ✅ COMPLETED |
| Plan 05 (`feature-prayer-calculation-1.md`) | PrayerTime entities, CalculateUseCase, adhan-dart, Hijri | ✅ COMPLETED |
| Plan 06 (`feature-prayer-cubit-1.md`) | PrayerTimeCubit, states, midnight timer | ✅ COMPLETED |
| Plan 07 (`feature-state-evaluation-1.md`) | DisplayState classes (5 states), EvaluateUseCase | ✅ COMPLETED |
| Plan 08 (`feature-display-state-machine-1.md`) | DisplayStateCubit, tick timer, power recovery | ✅ COMPLETED |
| Plan 09 (`feature-setup-wizard-logic-1.md`) | SetupWizardCubit, validation, step navigation | ✅ COMPLETED |
| Plan 10 (`feature-setup-wizard-ui-1.md`) | 4 Wizard Steps, City Picker, Preview, SplashPage, Widget Tests | ✅ COMPLETED |
| Plan 11 (`feature-settings-logic-1.md`) | SettingsCubit, auto-save, PIN management, Unit Tests | ✅ COMPLETED |
| Plan 12 | Settings UI | ✅ COMPLETED |

### DatabaseHelper — Pola Testing

Gunakan testing hooks berikut untuk unit test yang isolated:

```dart
setUpAll(() {
  sqfliteFfiInit();
  databaseFactory = databaseFactoryFfi;
});

final db = await databaseFactoryFfi.openDatabase(inMemoryDatabasePath);
final helper = DatabaseHelper();
await helper.createTablesForTesting(db);
await helper.insertDefaultSettingsForTesting(db);
helper.initForTesting(db);

tearDown(() async {
  await helper.close();
  helper.resetForTesting();
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GulajavaMinistudio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GulajavaMinistudio)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
