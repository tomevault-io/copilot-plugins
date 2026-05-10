## pingre-mobile

> This file provides guidance for AI assistants working in this codebase.

# CLAUDE.md — Pingre Mobile

This file provides guidance for AI assistants working in this codebase.

## Project Overview

**Pingre** is a cross-platform personal finance management app built with Flutter. It supports Android, iOS, Web, Windows, Linux, and macOS. The current state is UI/UX focused with in-memory fake data — no persistent database layer yet.

## Repository Layout

```
Pingre-mobile/
└── Pingre/               # Main Flutter project root
    ├── lib/
    │   ├── main.dart              # Entry point; initializes services and providers
    │   ├── theme_extensions.dart  # Custom semantic color theme extension
    │   ├── models/                # Pure data models
    │   ├── services/              # ChangeNotifier state managers (business logic)
    │   ├── screens/               # Full pages organized by feature
    │   └── widgets/               # Reusable UI components (data/, inputs/, layout/)
    ├── test/
    │   └── widget_test.dart       # Placeholder — no real tests yet
    ├── android/                   # Android platform config (Gradle, Kotlin, Java 17)
    ├── ios/                       # iOS platform config (XCode)
    ├── web/ windows/ linux/ macos/ # Desktop/web platform configs
    ├── pubspec.yaml               # Dependencies and version
    └── analysis_options.yaml      # Dart linter config (flutter_lints)
```

## Technology Stack

| Concern | Choice |
|---|---|
| Language | Dart (SDK ≥ 3.10.7) |
| Framework | Flutter |
| UI Design System | `forui` ^0.19.0 |
| State Management | `provider` ^6.1.5 (ChangeNotifier) |
| Currency Arithmetic | `decimal` ^3.2.4 |
| Persistence (settings only) | `shared_preferences` ^2.3.0 |
| ID generation | `uuid` ^4.5.3 |

## Architecture

### State Management

Uses the `provider` package with a `MultiProvider` at the root (`main.dart`). Services extend `ChangeNotifier` and call `notifyListeners()` on mutations.

- Read state reactively: `context.watch<FooService>()` (rebuilds on change)
- Read state once / trigger actions: `context.read<FooService>()`

### Services (`lib/services/`)

| File | Responsibility |
|---|---|
| `transactions.dart` | Transaction CRUD; generates 200 fake transactions on init |
| `accounts.dart` | Account CRUD; pre-populated with 7 test accounts |
| `tags.dart` | Tag CRUD; pre-populated with 18+ test tags |
| `recurring.dart` | Recurring transaction schedules; applies missed ones on startup |
| `settings.dart` | Theme mode + last recurring timestamp; backed by SharedPreferences |

All services are initialized in `main()` before `runApp()`. `SettingsService` must be explicitly `await`ed because it performs async `SharedPreferences` loading.

### Models (`lib/models/`)

- `TimeRange` — enum + factory constructors (`elapsed()`, `current()`) for period-based date logic
- `TransactionGroup` — groups transactions for display by time period
- `time_range_icon.dart` — icon mapping for time range values

### Screens (`lib/screens/`)

Feature folders mirror the bottom-nav tabs:

| Folder | Feature |
|---|---|
| `transactions/` | Transaction list, edit form, form fields |
| `accounts/` | Account list, edit, type icon helper |
| `recurring/` | Recurring list, edit |
| `reports/` | Report summary, filter bottom sheet |
| `tags/` | Tag list, edit, multi-select, display chip |
| `home_page.dart` | Root scaffold with bottom nav (5 tabs) |
| `app_settings_page.dart` | Theme selector and app preferences |

### Widgets (`lib/widgets/`)

| Subfolder | Contents |
|---|---|
| `data/` | `ValueDisplay`, `ErrorDisplay`, `ElasticPullRefresh` |
| `inputs/` | `ValueInput`, `TimeRangeSelect`, `ColorPicker`, `SearchAdd` |
| `layout/` | `SheetContainer` (modal sheet wrapper) |

### Navigation Pattern

- Bottom navigation bar (5 items: Transactions, Recurring, FAB add, Accounts, Reports)
- `MaterialPageRoute` push/pop for detail screens
- Bottom sheets (`showModalBottomSheet`) for edit forms

### Theming

- ForUI's `FTheme` wraps the app
- Light/dark/system modes stored via `SettingsService`
- Custom `AppSemanticColors` theme extension provides `positive` and `negative` colors
- Access via `Theme.of(context).extension<AppSemanticColors>()`

## Key Conventions

### Naming

- Files: `snake_case.dart`
- Classes: `PascalCase`
- Services: suffix `Service` (e.g., `TransactionsService`)
- Screens: suffix `Page` for full pages, `Edit` for edit forms/sheets

### Data Persistence

- **No database** currently — all transaction/account/tag data is lost on restart
- Only `SettingsService` persists data (via `SharedPreferences`)
- When adding persistence, follow the existing service pattern and add async loading in `main()`

### Decimal / Currency

Always use `Decimal` (from the `decimal` package) for monetary values, never `double`. This avoids floating-point rounding errors.

### Fake Data

`TransactionsService`, `AccountsService`, `TagsService`, and `RecurringTransactionsService` all initialize with fake/test data in their constructors. This is intentional for development; remove or gate it before production.

## Common Commands

Run from inside `Pingre/`:

```bash
flutter pub get          # Install/update dependencies
flutter analyze          # Run linter (must pass before committing)
flutter test             # Run tests (currently minimal)
flutter run              # Run in debug mode on connected device/emulator
flutter run --profile    # Profile mode
flutter run --release    # Release mode

# Platform builds
flutter build apk        # Android APK
flutter build ios        # iOS (requires macOS + Xcode)
flutter build web        # Web
flutter build linux      # Linux desktop
flutter build windows    # Windows desktop
flutter build macos      # macOS desktop
```

VSCode launch configs in `.vscode/launch.json` cover debug, profile, and release modes.

## Testing

Testing infrastructure is in place but effectively empty (`test/widget_test.dart` is a placeholder). When writing new tests:

- Place widget tests in `test/`
- Use `flutter_test` (already a dev dependency)
- Run with `flutter test`

## What's Not Here (Future Work)

- Persistent data storage (SQLite, Isar, Drift, or Supabase)
- Real authentication
- CI/CD pipelines
- Charts/graphs in the Reports tab (UI scaffolding exists)
- Actual production data — all current data is fake

---
> Source: [ndegheselle/Pingre-mobile](https://github.com/ndegheselle/Pingre-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
