## eco-hero

> This document provides a comprehensive overview of the Eco Hero project, intended as a guide for AI-driven development.

# Gemini Context: Eco Hero iOS App

This document provides a comprehensive overview of the Eco Hero project, intended as a guide for AI-driven development.

## Project Overview

Eco Hero is a modern iOS application designed to help users live a more environmentally friendly lifestyle. It's a gamified companion that allows users to scan waste for sorting advice, log eco-friendly activities, participate in challenges, and learn about sustainability.

### Technologies & Architecture

*   **Framework:** Swift & SwiftUI
*   **Target:** iOS (likely 17.0+ based on the use of `@Observable`)
*   **Architecture:** The project follows a clean, service-oriented architecture (MVVM-S).
    *   **UI:** SwiftUI is used for the entire user interface.
    *   **State Management:** It uses SwiftUI's latest `@Observable` framework for state management.
    *   **Data Persistence:** SwiftData is used for local on-device storage.
    *   **Backend Services:** The app is architected with services for authentication (`AuthenticationManager`) and data synchronization (`CloudSyncService`), implying a cloud backend, though the specific provider configuration is unconfirmed.
    *   **Machine Learning:** The app uses on-device machine learning for two key features:
        1.  **Waste Classification:** Uses the **Vision** and **Core ML** frameworks with the device's camera to classify waste in real-time.
        2.  **Eco Tips:** Includes a service (`TipModelService`) for providing tips, likely powered by a custom Core ML model.
*   **Dependencies:** The project appears to be self-contained, using native Apple frameworks without any external dependencies managed by SPM, CocoaPods, or Carthage.

### Core Features

The app is structured around five main tabs:

1.  **Home (`DashboardView`):** The main dashboard, likely displaying user progress and stats.
2.  **Log (`LogActivityView`):** Allows users to manually log their eco-friendly activities.
3.  **Challenges (`ChallengesView`):** A gamified section for users to complete challenges and earn achievements.
4.  **Sorter (`WasteSortingView`):** The core ML feature that uses the camera to help users sort waste.
5.  **Learn (`LearnView`):** A section for educational content on sustainability.

## Building and Running

*   **Environment:** Xcode (latest version recommended) is required.
*   **Setup:**
    1.  Clone the repository.
    2.  Open `Eco Hero.xcodeproj` in Xcode.
    3.  Dependencies are native and require no extra setup.
*   **Running the App:**
    *   Select the "Eco Hero" scheme.
    *   Choose an iOS Simulator (e.g., iPhone 15 Pro) or a connected physical device.
    *   Click the "Run" button (or press `Cmd+R`).
*   **Running Tests:**
    *   Open the Test Navigator in Xcode (`Cmd+6`).
    *   Click the "Play" button on the "Eco HeroTests" or "Eco HeroUITests" targets.
    *   Alternatively, use the command: `xcodebuild -scheme "Eco Hero" -sdk iphonesimulator test`

## Development Conventions

*   **Code Style:** The codebase uses modern, idiomatic Swift and SwiftUI. It heavily utilizes `async/await` for concurrency and follows best practices for separating concerns into views, models, and services.
*   **File Structure:** Code is organized by feature/type into folders like `Views`, `Models`, `Services`, `ViewModels`, etc.
*   **Machine Learning Workflow:**
    *   **TODO:** The project contains a `scripts/generate_tip_model.py` script, which is intended to train or convert a machine learning model for the eco-tips feature. The exact usage of this script is unknown as the file was inaccessible. Developers working on the tip recommender should investigate this script.
*   **Backend Configuration:**
    *   **TODO:** The backend provider (likely Firebase or iCloud) is not confirmed. Configuration for `CloudSyncService` and `AuthenticationManager` may be required. A `.firebaserc` file was detected but was inaccessible.

This document should be updated as more information about the `TODO` sections becomes available.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rishabhcli)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rishabhcli)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
