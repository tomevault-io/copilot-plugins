## tempo-timer

> This file contains guidelines for the Gemini AI assistant to follow when working on this project.

# Gemini Guidelines

This file contains guidelines for the Gemini AI assistant to follow when working on this project.

## Architecture

This project follows a Kotlin Multiplatform (KMP) architecture with a `shared` module for the core business logic and a `desktopApp` module for the desktop-specific implementation. The architecture is based on a variant of Clean Architecture, with a clear separation of concerns between the data, domain, and presentation layers.

### Key Technologies

*   **Kotlin Multiplatform:** For sharing code between different platforms.
*   **Jetpack Compose for Desktop:** For building the UI.
*   **Koin:** For dependency injection.
*   **Kotlin Coroutines:** For asynchronous programming.
*   **Ktorfit:** For type-safe HTTP clients.
*   **Room:** For local data persistence.

### Modules

*   **`shared`:** Contains the platform-independent code.
    *   **`commonMain`:** The main source set for the shared module.
        *   **`data`:** The data layer, including repositories, network services, and the database.
        *   **`feature`:** The presentation layer, with each feature in its own package.
        *   **`design`:** The design system, including colors, typography, and custom UI components.
*   **`desktopApp`:** Contains the desktop-specific implementation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matejsemancik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/matejsemancik)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
