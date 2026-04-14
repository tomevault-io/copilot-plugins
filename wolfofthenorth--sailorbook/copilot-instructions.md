## sailorbook

> This document provides context for the SailorBook project, a privacy-first reading and listening app for public-domain books.

# Gemini Project Context: SailorBook

This document provides context for the SailorBook project, a privacy-first reading and listening app for public-domain books.

## Project Overview

SailorBook is a cross-platform application built with Flutter for the UI and a Rust core for heavy lifting. The application allows users to download books from public sources, parse EPUBs locally, and generate human-like audio entirely on-device using local AI TTS.

**Key Technologies:**

*   **Frontend:** Flutter (Material 3, Riverpod, go_router)
*   **Backend (Core):** Rust (EPUB parsing, TTS synthesis with Piper/ONNX, audio caching)
*   **Testing:** Playwright for E2E tests, Flutter and Rust unit tests.

**Architecture:**

The application follows a two-layer architecture:

1.  **Flutter UI:** Handles all user interaction and state management.
2.  **Rust Core:** A library that is called from Flutter via FFI. It manages EPUB parsing, TTS synthesis, audio caching, and book manifest management.

## Building and Running

### Prerequisites

*   Flutter SDK 3.16.0+
*   Rust 1.70.0+
*   Node.js 18+ (for testing)

### Development Setup

1.  **Install Flutter dependencies:**
    ```bash
    cd apps/app_flutter
    flutter pub get
    ```

2.  **Build Rust core:**
    ```bash
    cd ../../crates/pdreader_core
    cargo build --release
    ```

3.  **Generate Flutter-Rust bridge:**
    ```bash
    cargo run --bin codegen
    ```

### Running the App

*   **Desktop (recommended):**
    ```bash
    cd apps/app_flutter
    flutter run -d macos  # or -d windows, -d linux
    ```
*   **Web:**
    ```bash
    flutter run -d web-server --web-port=3000
    ```

## Testing

*   **Flutter unit tests:**
    ```bash
    cd apps/app_flutter
    flutter test
    ```
*   **Rust unit tests:**
    ```bash
    cd ../../crates/pdreader_core
    cargo test
    ```
*   **Playwright E2E tests:**
    ```bash
    cd tests/playwright
    npm run test
    ```

## Development Conventions

*   **Code Style:** Use `dart format` and `cargo fmt` to format the code.
*   **Commits:** Use conventional commit messages.
*   **State Management:** Riverpod is used for state management in the Flutter application.
*   **UI:** The UI is built with Material 3.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WolfOfTheNorth)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WolfOfTheNorth)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
