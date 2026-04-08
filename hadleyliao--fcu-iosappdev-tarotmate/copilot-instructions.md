## fcu-iosappdev-tarotmate

> This is an iOS application for tarot card readings, built with SwiftUI. The project is in its early stages. The current Swift code is based on a default Xcode template, but there is a comprehensive design and prototype document (`doc/index.html`) that outlines the full vision for the app.

# Project: 神秘塔羅 (Mysterious Tarot) - iOS App

## Project Overview

This is an iOS application for tarot card readings, built with SwiftUI. The project is in its early stages. The current Swift code is based on a default Xcode template, but there is a comprehensive design and prototype document (`doc/index.html`) that outlines the full vision for the app.

The app's purpose is to allow users to perform daily tarot readings, explore different card spreads, browse a library of tarot cards, and keep a journal of their past readings.

**Key Technologies:**
*   **UI:** SwiftUI
*   **Data:** SwiftData (as indicated by `Tarot3App.swift`)
*   **Language:** Swift

**Architecture (based on `doc/index.html`):**

The application is designed around a `TabView` with four main sections:
1.  **Home (`HomeView.swift`):** The main screen for daily readings.
2.  **Card Library (`CardLibraryView.swift`):** A searchable gallery of all tarot cards.
3.  **Journal (`JournalView.swift`):** A log of past readings.
4.  **Settings (`SettingsView.swift`):** For app settings like appearance and notifications.

## Building and Running

This is an Xcode project. To build and run the application:

1.  Open `Tarot3.xcodeproj` in Xcode.
2.  Select an iOS simulator or a connected device.
3.  Click the "Run" button (or press `Cmd+R`).

*TODO: Add instructions for any specific dependencies or setup steps if they become necessary.*

## Development Conventions

The `doc/index.html` file serves as the primary design and development guide. It includes detailed screen mockups, navigation flows, and component specifications.

**Coding Style:**
*   The project uses standard Swift and SwiftUI conventions.
*   Views should be broken down into smaller, reusable components as suggested in the design document (e.g., `TarotCardView.swift`).
*   SwiftData is the chosen persistence framework. Models should be defined and used according to SwiftData best practices.

**Asset Management:**
*   Colors, icons, and images should be managed in `Tarot3/Assets.xcassets`.
*   The color palette and font styles defined in the HTML prototype should be implemented as assets for consistent use throughout the app.
*   Card artwork needs to be sourced and added to the assets. The current design uses placeholders.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hadleyliao)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hadleyliao)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
