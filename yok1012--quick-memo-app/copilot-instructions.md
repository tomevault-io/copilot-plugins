## quick-memo-app

> **quickMemoApp** is a comprehensive iOS and watchOS application designed for rapid note-taking. It features deep integration with the system Calendar (EventKit), a Widget extension, and robust Pro features managed via StoreKit 2 and CloudKit. The project focuses on speed and usability, specifically optimizing for Japanese input methods and offline-first usage on Apple Watch.

# GEMINI.md

## Project Overview
**quickMemoApp** is a comprehensive iOS and watchOS application designed for rapid note-taking. It features deep integration with the system Calendar (EventKit), a Widget extension, and robust Pro features managed via StoreKit 2 and CloudKit. The project focuses on speed and usability, specifically optimizing for Japanese input methods and offline-first usage on Apple Watch.

*   **Version:** 1.0 (Build 2)
*   **Status:** In App Store Review / Pre-release
*   **Platforms:** iOS 16.0+, watchOS 11.5+

## Technical Stack
*   **Language:** Swift 5.9
*   **UI Framework:** SwiftUI
*   **Storage:**
    *   Primary: `UserDefaults` (JSON encoded) within an App Group (`group.yokAppDev.quickMemoApp`)
    *   Secondary: CloudKit (for Pro user sync)
    *   Legacy/Future: Core Data stack exists but is currently unused.
*   **Connectivity:**
    *   `WatchConnectivity`: For syncing between iPhone and Apple Watch.
    *   `StoreKit 2`: In-app purchases (Subscription & One-time).
    *   `EventKit`: Calendar event creation.
*   **Localization:** Custom `LocalizationManager` for runtime language switching (Japanese, English, Simplified Chinese).

## Architecture
The application follows an **MVVM** pattern with a heavy reliance on **Singleton Services** for business logic and data management.

### Data Flow
1.  **User Action** (View) triggers a method in a **Service**.
2.  **Service** (e.g., `DataManager`) updates the in-memory model and persists to `UserDefaults`.
3.  **@Published** properties in the Service update the SwiftUI Views.
4.  **Sync:**
    *   **Watch:** `iOSWatchConnectivityManager` pushes data to the Watch. The Watch app is "offline-first" and queues data if the iPhone is unreachable.
    *   **Cloud:** `CloudKitManager` syncs data to iCloud if the user is a Pro subscriber.

### Key Services (`quickMemoApp/Services/`)
*   `DataManager`: Central CRUD for memos and categories.
*   `PurchaseManager`: Handles StoreKit transactions and Pro status entitlement.
*   `iOSWatchConnectivityManager` / `WatchConnectivityManager`: Manages bi-directional sync.
*   `NotificationManager`: Handles local notifications and quiet hours.
*   `LocalizationManager`: Manages app-specific language settings independent of the system locale.

## Key Features
*   **Quick Input:** Shake-to-undo style gesture to invoke a fast input view.
*   **Smart Tagging:** Automatic tag extraction (specialized for Japanese text).
*   **Calendar Sync:** Automatically creates calendar events from memos with duration support.
*   **Pro Features:**
    *   Unlimited memos/categories/tags.
    *   CloudKit Sync.
    *   Advanced customization.
*   **Widgets:** Configurable homescreen widgets showing recent memos per category.

## Development Setup

### Build Commands
*   **iOS Simulator:**
    ```bash
    xcodebuild -project quickMemoApp.xcodeproj -scheme quickMemoApp -configuration Release -destination 'platform=iOS Simulator,name=iPhone 17 Pro' build
    ```
*   **Watch Simulator:**
    ```bash
    xcodebuild -project quickMemoApp.xcodeproj -scheme "quickMemoWatch Watch App" -destination 'platform=watchOS Simulator,name=Apple Watch Series 10 (46mm)' build
    ```
*   **Tests:**
    ```bash
    xcodebuild test -project quickMemoApp.xcodeproj -scheme quickMemoApp -destination 'platform=iOS Simulator,name=iPhone 17 Pro'
    ```

### Debugging
*   **Logs:** Use `xcrun simctl spawn booted log stream --predicate 'subsystem contains "yokAppDev.quickMemoApp"'` to filter app logs.
*   **StoreKit:** The project uses `Configuration.storekit`. Ensure this file is referenced in the scheme for testing IAP flows.

## Project Structure
*   `quickMemoApp/`: Main iOS application code.
    *   `App.swift`: Entry point.
    *   `Models/`: `QuickMemo`, `Category` structs.
    *   `Views/`: SwiftUI views organized by feature.
    *   `Services/`: Business logic singletons.
    *   `Resources/`: Localizable strings.
*   `quickMemoWatch Watch App/`: Companion Watch app (highly independent logic).
*   `quickMemoWidget/`: Widget extension code.
*   `quickMemoAppTests/`: Unit tests.

## Conventions
*   **Style:** Swift 5.9, 4-space indentation.
*   **Naming:** `PascalCase` for types, `camelCase` for properties/methods.
*   **Commits:** Use `type: summary` format (e.g., `feat: Add calendar sync`).
*   **Localization:** Always use `LocalizationManager` or `Localizable.strings`. Do not hardcode strings in Views.

## Known Issues / Technical Debt
1.  **Storage Limit:** `UserDefaults` is used for main storage, which has a practical limit (~1MB). Migration to Core Data or SQLite will be necessary as user data grows.
2.  **Test Coverage:** Unit test coverage is currently low.
3.  **Watch Connectivity:** The sync architecture is asymmetric; the Watch is robust/offline-first, while the iOS side assumes connection.
4.  **Hardcoded Values:** Some category colors and UI constants are hardcoded and should be moved to a theme manager.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yok1012)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yok1012)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
