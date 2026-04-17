## moodlog

> MoodLog is a mood tracking and journaling Flutter app with the following key features:

# MoodLog Flutter Project - Gemini Guidelines

## Project Overview

MoodLog is a mood tracking and journaling Flutter app with the following key features:

- Local-first architecture (no login required)
- Mood tracking with AI-powered insights
- Private diary entries stored locally
- Statistics and analytics

## Project Structure

- **State Management**: Provider
- **Routing**: go_router
- **Database**: Drift (SQLite) for local storage
- **AI Integration**: Firebase AI (Gemini)
- **Analytics**: Firebase Analytics
- **Ads**: Google Mobile Ads

### Directory Guidelines: `services` vs. `utils`

To maintain a clean and scalable architecture, it's important to differentiate between `services`
and `utils`.

#### `core/services`

This directory contains classes that provide core functionalities or manage interactions with
external systems. They often have a lifecycle and maintain state.

- **Role**: Manages a specific app-wide service (e.g., logging, flavor management, ads).
- **State**: Often stateful and may be managed as singletons.
- **Lifecycle**: Typically initialized once when the app starts.
- **Examples**: `LoggingService`, `FlavorService`, `AdmobService`.

#### `core/utils`

This directory contains stateless helper functions or classes that perform common, reusable tasks.

- **Role**: Provides simple, reusable tools and helper functions (e.g., formatters, converters,
  keyboard management).
- **State**: Should be stateless.
- **Lifecycle**: No specific lifecycle; used on-demand.
- **Examples**: `ErrorHandler`, `KeyboardUtils`, `StringListConverter`.

## Architecture

### MVVM + Clean Architecture

```
lib/
├── core/           # Common utilities, constants
├── data/           # Repositories, data sources, models
├── domain/         # Entities, repository interfaces, use cases
└── presentation/   # Presentation Layer
    ├── screens/      # UI (View)
    ├── providers/    # State Management (ViewModel)
    ├── widgets/      # Common Widgets
    ├── theme/        # App themes
    └── ...
```

### File Patterns

- **Views (Screens)**: `lib/presentation/screens/*/*_view.dart`
- **ViewModels**: `lib/presentation/providers/*_view_model.dart` or
  `lib/presentation/screens/*/*_view_model.dart`
- **Widgets**: `lib/presentation/widgets/` or `lib/presentation/screens/*/widgets/`
- **Models**: `lib/data/models/`
- **Entities**: `lib/domain/entities/`

#### View File Structure (`part`/`part of`)

To ensure consistency and separation of concerns within the presentation layer, all view files
should follow the `part`/`part of` convention:

- **`*_view.dart`**: This file should contain only one `StatelessWidget` (the main screen widget)
  which is responsible for providing the `ViewModel` to the widget tree.
- **`*_content.dart`**: This file, marked with `part of '*_view.dart'`, contains the `_Content`
  widget which builds the actual UI of the screen.

This structure separates the DI setup from the UI implementation, making the code cleaner and easier
to navigate.

## Key Architectural Decisions (v1.0.28+)

### Local-First Architecture

**No Authentication Required**:

- Firebase Auth has been removed
- Users are created locally on first app launch
- All data is stored locally (Drift + SharedPreferences)

**User Management**:

- `LocalUser` entity with: userId (UUID), nickname, profileImagePath, createdAt
- Stored in SharedPreferences
- No email or external account linking

**Onboarding Flow**:

1. First launch → Show onboarding screens
2. User sets nickname and AI personality
3. Local user automatically created
4. App ready to use

**Data Storage**:

- Journal entries: Drift (SQLite)
- User info & settings: SharedPreferences
- No cloud sync (future feature)

## Development Conventions

### Internationalization

- **Supported Languages**: Korean (default), English, Japanese
- **Translation Files**: `lib/core/l10n/app_{language_code}.arb`
- Only edit `.arb` files; `.dart` files are auto-generated
- Base language is Korean (ko)

### Code Generation

```bash
# Generate localization files
flutter gen-l10n

# Generate freezed/json_serializable code
dart run build_runner build --delete-conflicting-outputs
```

### Common Commands

```bash
flutter analyze          # Lint check
flutter test            # Run tests
flutter build apk       # Build Android APK
```

### Code Style

- Use `withValues(alpha: ..)` instead of deprecated `withOpacity`
- Follow MVVM pattern: View → ViewModel → UseCase → Repository
- Use freezed for immutable entities and models
- Use Provider for state management
- **Avoid unnecessary comments**: Write self-explanatory code. Only add comments for complex logic, important business rules, or non-obvious implementations

### Freezed Usage Convention

When creating a class using the `freezed` package, it must be defined as an `abstract class` with a `mixin`. Using a plain class can lead to unexpected build errors.

```dart
// ✅ Good
@freezed
abstract class User with _$User {
  const factory User({
    required String name,
    required int age,
  }) = _User;
}

// ❌ Avoid
@freezed
class User with _$User {
  const factory User({
    required String name,
    required int age,
  }) = _User;
}
```

### Provider Usage Convention

When using Provider for state management, prefer `context.select` over `context.watch` to optimize widget rebuilds:

```dart
// ✅ Good - Only rebuilds when isLoading changes
final isLoading = context.select((HomeViewModel vm) => vm.isLoading);

// ❌ Avoid - Rebuilds on any ViewModel change
final viewModel = context.watch<HomeViewModel>();
final isLoading = viewModel.isLoading;
```

**Guidelines**:
- Use `context.select((ViewModelType vm) => vm.property)` for reading specific properties
- Use `context.read<ViewModelType>()` for calling methods (actions)
- Only use `context.watch<ViewModelType>()` when you need the entire ViewModel instance

### Logging Conventions

- **`presentation` Layer**: Use `debugPrint()` for UI-related, temporary debugging only. Avoid
  committing `debugPrint` statements.
- **Other Layers (`core`, `data`, `domain`)**: Use the `logging` package. Create a `Logger`
  instance (`final Logger _log = Logger('ClassName');`) and use its methods (`_log.info`,
  `_log.warning`, `_log.severe`) for structured logging.

### Key Components

- **`UnifiedCalendarWidget`**: A stateless widget on the home screen that displays a horizontal list for date selection. The expanded grid view is now shown in a `CalendarBottomSheet`.

### User Management

- `LocalUser`: User entity (domain/entities/user/)
- `LocalUserRepository`: User data management (domain/repositories/)
- `LocalUserRepositoryImpl`: SharedPreferences implementation (data/repositories/)
- `UserProvider`: User state management (presentation/providers/)

### Settings Management

- `Settings`: App settings entity
- `AppStateProvider`: Settings state management
- Stored in SharedPreferences

### Journal Management

- `Journal`: Journal entry entity
- `JournalRepository`: Journal CRUD operations
- `JournalLocalDataSource`: Drift database operations
- Stored in SQLite

### AI Integration

- Firebase AI (Gemini) for mood analysis
- `GeminiRepository`: AI operations
- `GeminiUseCase`: Business logic for AI features

## Important Notes for AI Assistants

1. **Do NOT reintroduce Firebase Auth** - The app is now local-first by design
2. **Do NOT add login/signup screens** - Use onboarding only
3. **LocalUser is NOT Firebase User** - Different entity with different fields
4. **No email field** - LocalUser only has userId, nickname, profileImagePath, createdAt
5. **All data is local** - Cloud backup/restore is a manual user-triggered action, not automatic sync.
6. **App Lock and Cloud Backup/Restore are FREE features.** Do not attempt to re-implement them as premium features.
7. **All design documents must be written in Korean.**

## Release Management

### Patch Notes

- Location: `docs/patch/{version}`
- Languages: ko-KR, en-US, ja-JP
- Format: Use emoji categories (🌍 new feature, 🎨 improvement, 🔧 bug fix)
- Length: Max 30 characters per language

### Version Bump

Always ask before bumping version in `pubspec.yaml`

## Reference Documents

- Architecture Plan: `docs/local_first_architecture_plan.md`
- Refactoring Plan: `docs/REFACTORING_PLAN.md`
- Code Analysis: `docs/code_analysis_report.md`

## Supported Languages

- Korean (한국어) - Default
- English
- Japanese (日本語)

---

## Recent Changes

- **`home_content` Refactoring**: Refactored the `_HomeScreenContent` widget by separating the
  `AppBar` and `FloatingActionButton` into their own widgets to improve code readability and
  maintainability.
- **Skeleton UI Dark Mode Fix**: Modified the `Skeleton` widget to use theme colors instead of
  hardcoded ones, fixing an issue where the UI appeared too bright in dark mode.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/log-mind-365) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
