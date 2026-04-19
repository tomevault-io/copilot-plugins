## queens

> - **Framework:** Flutter

# Queens Project Rules (.cursor-rules)

## Overview
- **Framework:** Flutter
- **SDK Constraint:** ^3.5.0
- **State Management:** flutter_bloc (^8.1.6)
- **Navigation:** go_router (^14.8.1)
- **Linting:** very_good_analysis (^6.0.0)
- **Code Generation:** Build Runner (implied by `flutter: generate: true`)

## Project Structure & Naming Conventions

### Folder Structure
- **`lib/`**: Root for Dart code.
  - `main_*.dart`: Environment-specific entry points.
  - `bootstrap.dart`: Common app initialization.
  - **`app/`**: Core app components (root widget, routing).
    - `view/`: Root widget (`App`).
    - `router/`: Navigation configuration (`app_router.dart`).
  - **`theme/`**: Theming (`app_theme.dart`).
  - **`l10n/`**: Localization.
  - **`<feature_name>/`**: Feature modules (e.g., `counter/`).
    - `<feature_name>.dart`: Feature export file (optional but common).
    - `view/`: Feature-specific UI widgets/pages.
    - `cubit/` or `bloc/`: Feature-specific state management.
    - (Potentially `models/`, `widgets/`, `data/` as needed within features).
- **`test/`**: Automated tests.

### Naming Conventions
- **Files & Directories:** `snake_case` (e.g., `app_theme.dart`, `feature_module/`).
- **Classes:** `PascalCase` (e.g., `MyWidget`, `AppTheme`, `CounterCubit`).
- **Functions, Methods, Variables:** `camelCase` (e.g., `getUser`, `build`, `userName`).
- **Constants:** `camelCase` (preferred, e.g., `router`, `AppRoutes.home`) or `kPrefixedCamelCase` (e.g., `kDefaultPadding`) for global constants.

## Key Technical Decisions & Conventions

### Theming
- **System:** Material 3 (via `ThemeData(useMaterial3: true)`).
- **Color Schemes:** Uses generated color schemes from `MaterialTheme` in `lib/theme/app_theme.dart`.
- **Modes:** Supports light and dark modes, synced with system settings (`ThemeMode.system`).
- **Contrast:** High/Medium contrast color schemes are defined in `MaterialTheme` but are **not** currently selectable or automatically applied.
- **Typography:** Uses Google Fonts ("Open Sans" for body, "Adamina" for display) via the `google_fonts` package and a custom `createTextTheme` function in `lib/theme/app_theme.dart`.
- **Color Usage:** Always use theme colors from `Theme.of(context).colorScheme` or other theme properties instead of hardcoding colors. If custom colors are necessary, document the reason and consider adding them to the theme system instead.
  - **✅ Do:** `backgroundColor: theme.colorScheme.primary`
  - **❌ Don't:** `backgroundColor: Colors.purple`
  - If a custom color is needed, add a comment explaining why and consider defining it in the theme.

### Navigation
- **Method:** Declarative routing using `go_router`. Configuration in `lib/app/router/app_router.dart`.
- **Initial Route:** Set via `GoRouter.initialLocation`.
- **Recommendation:** Define route paths as constants (e.g., in `AppRoutes` class). Leverage `go_router` features like named routes, parameters, and shell routes as needed.

### State Management
- **Primary:** `flutter_bloc` / `bloc`.
- **Convention:** Features (like `counter`) should contain their own Blocs/Cubits.
- **Observation:** `AppBlocObserver` is set up in `bootstrap.dart` for logging state changes and errors.

### Dependencies
- **Core:** `flutter`, `bloc`, `flutter_bloc`
- **Navigation:** `go_router`
- **UI/Utils:** `google_fonts`, `intl`
- **Testing:** `flutter_test`, `bloc_test`, `mocktail`

## Best Practices Checklist/Recommendations
- **[DONE]** Centralize theme definitions.
- **[DONE]** Use `flutter_bloc` for state management within features.
- **[DONE]** Structure code by feature.
- **[DONE]** Utilize `very_good_analysis` for strict linting.
- **[DONE]** Use a declarative navigation package (`go_router`).
- **[CONSIDER]** Implement user-selectable contrast themes using the defined schemes in `MaterialTheme` and state management.
- **[DEFERRED]** Migrate to a declarative navigation package (e.g., `go_router`) as the app grows.
- **[TODO]** Add specific component themes (Buttons, Cards, TextFields, etc.) to `ThemeData` in `lib/theme/app_theme.dart` for a more complete look and feel consistent with the color scheme.
- **[TODO]** Ensure `flutter pub get` is run after `pubspec.yaml` changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fgbytes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
