## playcado

> This document provides a comprehensive overview of the Playcado project, its structure, and development conventions.

# Playcado Project Overview

This document provides a comprehensive overview of the Playcado project, its structure, and development conventions.

## Project Overview

Playcado is a high-performance, lightweight Jellyfin client built with Flutter for iOS and Android. It focuses on a smooth user experience, hardware-accelerated playback, and robust offline media management.

### Core Technologies

*   **Framework:** [Flutter 3.8.0+](https://flutter.dev/)
*   **State Management:** [flutter_bloc](https://bloclibrary.dev/) (using the BLoC pattern)
*   **API Client:** [jellyfin_dart](https://github.com/JchrisM12/jellyfin-dart) (custom fork based on [devaryakjha/jellyfin-dart](https://github.com/devaryakjha/jellyfin-dart)) & [Dio](https://pub.dev/packages/dio)
*   **Navigation:** [go_router](https://pub.dev/packages/go_router)
*   **Video Engine:** [media_kit](https://pub.dev/packages/media_kit) (mpv-based)
*   **Downloads:** [background_downloader](https://pub.dev/packages/background_downloader)
*   **Casting:** [flutter_chrome_cast](https://pub.dev/packages/flutter_chrome_cast)
*   **Persistence:** [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage) & custom JSON-based `PreferencesService`
*   **Error Tracking:** [Sentry](https://sentry.io/)

### Architecture

The application follows a feature-first architecture. Each domain or major functionality resides in its own top-level directory within `lib/`.

*   **`lib/bootstrap.dart`**: The initialization core. Handles service registration, logging setup, and auto-login logic before the app starts.
*   **`lib/app_router/`**: Centralized navigation logic using `StatefulShellRoute` for persistent bottom navigation.
*   **`lib/auth/`**: Authentication logic, session management, and credential handling.
*   **`lib/media/`**: Core data layer for Jellyfin items, containing repositories, remote data sources, and the URL generation logic.
*   **`lib/video_player/`**: A unified playback system that handles both local (media_kit) and remote (Google Cast) playback states.
*   **`lib/downloads/`**: Background download management and local file persistence.
*   **`lib/theme/`**: Dynamic Material 3 theming engine with support for custom seed colors.
*   **`lib/widgets/`**: Common UI components used across the application.

## Building and Running

### Prerequisites

*   Flutter SDK: `^3.8.0`
*   FVM (optional, but recommended)
*   Jellyfin Server: v10.8.0 or higher

### Running the App

The project uses Flutter flavors (`dev`, `staging`, `prod`).

1.  **Install dependencies:**
    ```bash
    flutter pub get
    ```

2.  **Generate code (JSON Serializers):**
    ```bash
    dart run build_runner build --delete-conflicting-outputs
    ```

3.  **Run with flavor:**
    ```bash
    flutter run --flavor dev
    ```

## Development Conventions

### State Management & Async Data

*   **BLoCs:** Every feature uses BLoC for business logic.
*   **StatusWrapper:** Most states use a `StatusWrapper<T>` (found in `lib/status_wrapper/`) to standardize the handling of `initial`, `loading`, `success`, and `error` states.
*   **Repository Pattern:** UI components never talk to Data Sources directly. They interact with Repositories provided via `RepositoryProvider`.

### Logging

Use the centralized `LoggerService` (found in `lib/logger/`) for all console output. Avoid `print` or `debugPrint` directly in business logic.
```dart
LoggerService.auth.info('User logged in');
```

### Documentation & Comments

*   **DartDoc (`///`)**: Used for public API documentation (classes and methods).
*   **Inline Comments (`//`)**: Used only for complex logic or specific workarounds.

### Internationalization

Playcado uses ARB files for localization.
*   Template: `lib/l10n/app_en.arb`
*   Generated files: `lib/l10n/app_localizations.dart`

### Routing & Navigation

Navigation is handled via `context.go()` or `context.push()`. Redirection logic (Auth guards and Onboarding checks) is centralized in `lib/app_router/app_router.dart`.

## Secrets Handling

Playcado uses a compile-time injection strategy for secrets (like Sentry DSN) using the `--dart-define-from-file` flag.

*   **Logic:** Secrets are accessed via `lib/core/secrets.dart` using `String.fromEnvironment`.
*   **Security:** Actual secret values are stored in `config/secrets_*.json` (e.g., `config/secrets_dev.json`), which are ignored by Git.
*   **Workflow:**
    1.  Copy `config/secrets_dev.json.example` to `config/secrets_dev.json`.
    2.  Fill in the required values.
    3.  Run the app with `--dart-define-from-file=config/secrets_dev.json`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JchrisM12)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JchrisM12)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
