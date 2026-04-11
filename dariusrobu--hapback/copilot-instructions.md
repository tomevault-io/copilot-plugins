## hapback

> **Hapback** is a native iOS/macOS application built using **Swift** and **SwiftUI**. It leverages **SwiftData** for local data persistence.

# Hapback - Project Overview

## 1. Project Description
**Hapback** is a native iOS/macOS application built using **Swift** and **SwiftUI**. It leverages **SwiftData** for local data persistence.

The project currently follows the standard Xcode template structure for a SwiftUI app with SwiftData integration, featuring a basic list interface to add, view, and delete timestamped items.

## 2. Technology Stack
*   **Language:** Swift
*   **UI Framework:** SwiftUI
*   **Data Persistence:** SwiftData (`@Model`, `ModelContext`)
*   **IDE:** Xcode

## 3. Key Files & Structure
*   **`Hapback/HapbackApp.swift`**: The application entry point (`@main`). It initializes the `ModelContainer` for SwiftData and sets up the root `ContentView`.
*   **`Hapback/ContentView.swift`**: The primary user interface. It utilizes `NavigationSplitView` for layout and `@Query` to fetch and display `Item` records from the data store.
*   **`Hapback/Item.swift`**: The data model definition. A simple SwiftData class marked with `@Model`.
*   **`Hapback/Assets.xcassets`**: detailed resource catalog for colors and icons.

## 4. Development & Usage
### Prerequisites
*   macOS with the latest version of **Xcode**.

### Building and Running
This project is an Xcode workspace/project.
1.  Open `Hapback/Hapback.xcodeproj` in Xcode.
2.  Select the target simulator or connected device.
3.  Press `Cmd + R` to build and run.

### Testing
*   **Unit Tests:** Located in `HapbackTests/`.
*   **UI Tests:** Located in `HapbackUITests/`.
*   Run tests via Xcode using `Cmd + U`.

## 5. Coding Conventions
*   **SwiftUI:** Use declarative UI patterns.
*   **Persistence:** Use SwiftData's `@Model` for entities.
    *   Inject the container via `.modelContainer(sharedModelContainer)` in the App struct.
    *   Access the context via `@Environment(\.modelContext)`.
    *   Use `@Query` for auto-updating views based on data changes.
*   **Concurrency:** Stick to MainActor for UI updates; use standard Swift concurrency (`async`/`await`) if complex background tasks are introduced.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dariusrobu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dariusrobu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
