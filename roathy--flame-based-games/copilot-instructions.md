## flame-based-games

> This is a Flutter mobile application that serves as a host for several mini-games built using the **Flame engine**. The project is well-structured, following **Clean Architecture** principles to separate concerns.

# Gemini Project Context: flame_based_games

## 1. Project Overview

This is a Flutter mobile application that serves as a host for several mini-games built using the **Flame engine**. The project is well-structured, following **Clean Architecture** principles to separate concerns.

- **Project Type:** Flutter App (iOS, Android, Web, etc.)
- **Primary Frameworks/Libraries:**
    - **Game Engine:** [Flame](https://flame-engine.org/)
    - **State Management:** [Riverpod](https://riverpod.dev/)
    - **Routing:** [GoRouter](https://pub.dev/packages/go_router)
    - **Dependency Injection:** [GetIt](https://pub.dev/packages/get_it)
    - **Networking:** [Dio](https://pub.dev/packages/dio)

## 2. Architecture & Conventions

The codebase is organized by features (e.g., `raining_words_game`, `bouncing_words_game`) and follows a clear architectural pattern.

- **Clean Architecture:** The project separates code into three main layers:
    1.  **Data Layer:** (`/data`) Contains data sources (local or remote) and repository implementations.
    2.  **Domain Layer:** (`/domain`) Contains business logic, including repository contracts (interfaces) and use cases.
    3.  **Presentation/UI Layer:** (`/presentation` or `/flame`) Contains Flutter widgets and Flame game components.

- **Dependency Injection:** The `get_it` package is used as a service locator to provide dependencies (like repositories and use cases) throughout the app. The setup is located in `lib/core/di/injection_container.dart`.

- **State Management:** `flutter_riverpod` is used for managing state, especially in the UI layer (outside of the Flame game loop).

- **Routing:** Navigation is handled by `go_router`, with routes defined in `lib/core/router/app_router.dart`. The app uses a generic `FlameGameHostPage` to load different game levels, identified by a `levelId`.

- **Code Style:** The project uses the standard `flutter_lints` package to enforce consistent style and best practices.

## 3. How to Build and Run

### Prerequisites
- Flutter SDK installed.
- An available device or emulator.

### Key Commands

1.  **Get Dependencies:**
    ```bash
    flutter pub get
    ```

2.  **Run the App (Development):**
    ```bash
    flutter run
    ```

3.  **Run Tests:**
    ```bash
    flutter test
    ```

4.  **Analyze Code:**
    ```bash
    flutter analyze
    ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Roathy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Roathy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
