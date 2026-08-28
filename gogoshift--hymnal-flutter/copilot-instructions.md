## hymnal-flutter

> This file contains guidelines and commands for agentic coding agents working on Flutter projects.

# AGENTS.md

This file contains guidelines and commands for agentic coding agents working on Flutter projects.

## Project Overview

This is a Flutter mobile application using Clean Architecture with GetIt for dependency injection. The app follows a layered architecture pattern with clear separation of concerns.

## Development Environment Setup

**Required Flutter Version:** 3.38.9 (managed via FVM)

```bash
# Install Flutter version specified in .fvmrc
fvm install

# Set Flutter version for VS Code
fvm use 3.38.9

# Run the app
fvm flutter run
```

## Essential Commands

### Build and Run

```bash
# Install dependencies
fvm flutter pub get

# Run the app in debug mode
fvm flutter run

# Build for release
fvm flutter build apk --release
fvm flutter build ios --release
```

### Testing

```bash
# Run all tests
fvm flutter test

# Run a specific test file
fvm flutter test test/widget_test.dart

# Run tests with coverage
fvm flutter test --coverage

# Run tests on specific platform
fvm flutter test --platform android
```

### Code Quality

```bash
# Analyze code for issues
fvm flutter analyze

# Format code
fvm dart format .

# Generate localization files
fvm flutter gen-l10n

# Generate app icons (after pubspec changes)
fvm flutter pub run flutter_launcher_icons
```

## Architecture Guidelines

### Project Structure

```
lib/
├── constants/           # App-wide constants
├── services/           # Global services (locator, localization)
├── styles/             # Theme, fonts, sizes, assets
├── utils/              # Utility functions and mixins
├── widgets/            # Reusable UI components
└── layers/             # Clean architecture layers
    ├── data/           # Data sources, repositories, DTOs
    ├── domain/         # Use cases, models, business logic
    └── screens/        # UI screens and controllers
```

### Key Architectural Patterns

1. **Clean Architecture**: Three-layer separation (data, domain, screens)
2. **Dependency Injection**: GetIt service locator pattern
3. **State Management**: GetX with GetItMixin for dependency injection in widgets
4. **Controller Pattern**: Screens use `part` directive to separate controller logic

### Screen Structure

Each screen should follow this pattern:

- Screen file contains the UI widget
- Controller logic is in a separate file using `part` directive
- Controllers extend `ScreenBase<T>` for common functionality

Example:

```dart
// main_screen.dart
part 'main_controller.dart';

class MainScreen extends StatefulWidget with GetItStatefulWidgetMixin {
  // UI implementation
}

class _MainScreenState extends MainController {
  // State and UI methods
}

// main_controller.dart
part of 'main_screen.dart';

abstract class MainController extends ScreenBase<MainScreen> {
  // Business logic and state management
}
```

## Code Style Guidelines

### Naming Conventions

- **Files**: snake_case (e.g., `main_screen.dart`, `user_service.dart`)
- **Classes**: PascalCase (e.g., `MainScreen`, `UserService`)
- **Variables/Methods**: camelCase (e.g., `userName`, `getUserData()`)
- **Constants**: SCREAMING_SNAKE_CASE (e.g., `API_BASE_URL`)
- **Private members**: Prefix with underscore (e.g., `_privateMethod`)

### Import Organization

1. Flutter/Dart SDK imports
2. Package imports (sorted alphabetically)
3. Project imports (sorted by path depth)

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:get_it_mixin/get_it_mixin.dart';

import 'package:aaaf_mobile/services/locator_service.dart';
import 'package:aaaf_mobile/styles/font_sizes.dart';
import 'package:aaaf_mobile/widgets/screen_base.dart';

import 'package:aaaf_mobile/layers/data/dto/user.dart';
import 'package:aaaf_mobile/layers/domain/use_case/get_user_use_case.dart';
```

### Error Handling

- Use `try-catch` blocks for async operations
- Log errors in debug mode using `if (kDebugMode) { print(error); }`
- Use Either pattern with `dartz` package for operations that can fail
- Return appropriate error states to UI layers

### Widget Guidelines

- Use `const` constructors where possible
- Prefer `StatelessWidget` over `StatefulWidget` when state is not needed
- Use `GetItStatefulWidgetMixin` for dependency injection in stateful widgets
- Separate business logic into controller classes

### Dependency Injection

- All services are registered in `locator_service.dart`
- Use `get<T>()` to resolve dependencies in controllers
- Register services with appropriate lifetime:
  - `registerSingleton` for single-instance services
  - `registerLazySingleton` for lazy-loaded singletons
  - `registerFactory` for new instances each time

## Testing Guidelines

### Test Structure

- Unit tests: `test/unit/` (not yet created)
- Widget tests: `test/widget_test.dart`
- Integration tests: `integration_test/` (not yet created)

### Test Setup

Always initialize the service locator in test `setUp()`:

```dart
setUp(() {
  initializeGetIt();
});
```

### Running Tests

- Use `fvm flutter test` to run all tests
- Use specific test file path to run single test
- Tests should cover use cases and critical business logic

## Localization

- Use `flutter_gen/gen_l10n/app_localizations.dart` for localized strings
- Access strings via `AppLocalizations.of(context)!.key_name`
- Run `fvm flutter gen-l10n` after updating localization files
- Configuration is in `l10n.yaml`

## Assets and Styling

### Images

- SVGs: `assets/svgs/` (use `flutter_svg` package)
- PNG images: `assets/images/`
- Access via constants in `styles/svg_images.dart` and `styles/images.dart`

### Fonts

- Primary: Open Sans (multiple weights)
- Secondary: Nunito (multiple weights)
- Font sizes defined in `styles/font_sizes.dart`
- Font families in `styles/font_families.dart`

### Theme

- App theme configuration in `styles/theme.dart`
- Design tokens in `styles/design_styles.dart`
- Sizes and spacing in `styles/design_sizes.dart`

## HTTP and API

- Use `dio` package for HTTP requests
- API classes in `layers/data/source/remote/`
- Repository pattern for data access
- DTOs in `layers/data/dto/`

## Security

- Use `flutter_secure_storage` for sensitive data
- Secure storage wrapper in `layers/data/source/local/secure_storage.dart`
- Never commit API keys or sensitive configuration

## Common Patterns

### Use Case Pattern

```dart
class GetUserUseCase {
  Future<Either<Failure, User>> call() async {
    // Implementation
  }
}
```

### Repository Pattern

```dart
abstract class UserRepository {
  Future<Either<Failure, User>> getUser();
}

class UserRepositoryImpl implements UserRepository {
  // Implementation with data sources
}
```

### Screen with GetIt

```dart
class MyScreen extends StatefulWidget with GetItStatefulWidgetMixin {
  @override
  State<MyScreen> createState() => _MyScreenState();
}

class _MyScreenState extends MyScreenController {
  @override
  Widget build(BuildContext context) {
    final user = watchOnly((UserService s) => s.currentUser);
    // UI implementation
  }
}

// in a different file
abstract class MyController extends ScreenBase<MyScreen> {
  // Business logic and state management
}
```

## Development Tips

- Use FVM for Flutter version management
- Always run `flutter analyze` before committing
- Check pubspec.yaml for available scripts and dependencies
- Use the PageStorageKey for preserving scroll state
- Follow the established folder structure when adding new features
- Check existing files for patterns before creating new ones

---
> Source: [GoGoShift/Hymnal-Flutter](https://github.com/GoGoShift/Hymnal-Flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
