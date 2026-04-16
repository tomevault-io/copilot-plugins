## caffeine-tracker-in-class

> This document provides an overview of the `caffeine_tracker` Flutter project, detailing its purpose, technologies, and development conventions.

# GEMINI.md

This document provides an overview of the `caffeine_tracker` Flutter project, detailing its purpose, technologies, and development conventions.

## Project Overview

The `caffeine_tracker` project is a newly initialized Flutter application. While the project name suggests an intention to track caffeine intake, the current codebase represents the default Flutter counter application. It serves as a starting point for developing a cross-platform mobile application.

**Key Technologies:**
*   **Flutter:** UI toolkit for building natively compiled applications for mobile, web, and desktop from a single codebase.
*   **Dart:** Programming language used by Flutter.
*   **Dart SDK:** ^3.11.0

## Building and Running

To get started with the project, ensure you have Flutter installed and configured.

1.  **Get Dependencies:**
    ```bash
    flutter pub get
    ```

2.  **Run the Application:**
    ```bash
    flutter run
    ```
    To specify a device (e.g., Chrome, Android emulator), use `flutter run -d <device_id>`.

3.  **Run Tests:**
    ```bash
    flutter test
    ```

## Development Conventions

*   **Linting:** The project adheres to the recommended Flutter linting rules provided by `package:flutter_lints/flutter.yaml`. This ensures good coding practices and consistency across the codebase.
*   **Testing:** Widget testing is performed using the `flutter_test` package, following standard Flutter testing patterns as demonstrated in `test/widget_test.dart`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edharcourt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
