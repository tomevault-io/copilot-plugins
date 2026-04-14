## karoo-battery-estimator

> This project is an Android application designed as an extension for the Hammerhead Karoo cycling computer. Its primary purpose is to estimate how much time left of using Karoo while cycling.

# Karoo Camera Control

## Project Overview

This project is an Android application designed as an extension for the Hammerhead Karoo cycling computer. Its primary purpose is to estimate how much time left of using Karoo while cycling.

The project is built using:
-   **Language:** Kotlin
-   **UI Framework:** Jetpack Compose (Material3)
-   **Build System:** Gradle (Kotlin DSL)
-   **Key Library:** `io.hammerhead:karoo-ext` (Hammerhead Karoo Extension SDK)

## Executive Summary

A Karoo extension that predicts remaining battery life in both hours and kilometers, helping cyclists avoid running out of battery during long rides. This addresses a common pain point, especially for gravel/bikepacking riders on multi-hour adventures.

### Key Value Proposition

- Real-time battery predictions: "6.2h remaining / 124km at current pace"
- Intelligent warnings before battery depletion
- Learning from historical ride patterns
- Route-aware predictions

More docs available in the @docs/ folder.

### Hammerhead Karoo integration

When implementing Hammerhead Karoo functionality please follow guidelines and code samples from the following official Karoo sdk repository: https://github.com/hammerheadnav/karoo-ext. Especially different data sources, types and extensions are available here: https://github.com/hammerheadnav/karoo-ext/tree/master/app/src/main/kotlin/io/hammerhead/sampleext/extension.

## Building and Running

### Build Commands

**Build the project:**

    ```bash
    ./gradlew build
    ```

## Project Structure

- **`app/`**: The main application module.

## Development Conventions

- **Jetpack Compose:** The UI is built using Jetpack Compose.
- **Kotlin DSL:** Gradle build scripts are written in Kotlin (`.kts`).
- **Namespace:** `com.karoobatteryestimator`
- Keeping project aligned with DRY and clean code rules.
- Separate the business logic with app screens.
- Keep the code logically separated. Each functionality should have it's own class.
- All strings should not be kept in the code, but in resource files, allowing translations.

## Additional rules

- Do not commit the code unless stated clearly to do so.
- Plan first, ask for approval and then implement.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michalchecinski)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/michalchecinski)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
