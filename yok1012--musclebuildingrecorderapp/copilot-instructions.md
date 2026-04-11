## musclebuildingrecorderapp

> This file serves as the primary context and instruction manual for AI agents (Gemini) interacting with the `MuscleBuildingRecorder` project. It aggregates key architectural details, development conventions, and operational commands to ensure safe and efficient assistance.

# GEMINI.md

This file serves as the primary context and instruction manual for AI agents (Gemini) interacting with the `MuscleBuildingRecorder` project. It aggregates key architectural details, development conventions, and operational commands to ensure safe and efficient assistance.

## 1. Project Overview

**Name:** MuscleBuildingRecorder (筋トレ記録アプリ)
**Type:** Native iOS & watchOS Application
**Primary Language:** Swift 5.9+
**Frameworks:** SwiftUI, HealthKit, WatchConnectivity, CoreMotion, ActivityKit, Core Data, Combine.

**Purpose:**
A comprehensive workout tracking application that synchronizes data between an iPhone and Apple Watch. It features:
-   **Timer:** Work/Rest interval timer.
-   **Heart Rate Monitoring:** Real-time tracking via HealthKit (Apple Watch), BLE devices, and AirPods Pro.
-   **Sensor Data:** Collection of accelerometer and gyroscope data from Apple Watch (up to 100Hz).
-   **Synchronization:** Real-time state and data sync between iPhone and Watch using `WatchConnectivity`.
-   **Data Management:** Core Data for persistence, CSV/JSON export capabilities.
-   **UI:** SwiftUI-based, supporting Dynamic Island (Live Activities).

## 2. Architecture & Directory Structure

The project follows a standard Xcode project structure with separated targets for iOS and watchOS, plus a Shared module.

```
MuscleBuildingRecorder/
├── MuscleBuildingRecorder/              # iOS Application Target
│   ├── MuscleBuildingRecorderApp.swift  # Entry Point
│   ├── ContentView.swift                # Main View
│   ├── Models/                          # Core Data & Domain Models
│   ├── ViewModels/                      # Session & State Management
│   ├── Views/                           # SwiftUI Views
│   ├── Services/                        # HealthKit, BLE, HeartRate Services
│   ├── Utils/                           # Exporters, Helpers
│   ├── LiveActivityManager.swift        # Dynamic Island/Live Activity Logic
│   └── Resources/                       # Assets.xcassets, Info.plist
│
├── MuscleBuildingRecorderWatchTrue Watch App/ # watchOS Application Target
│   ├── MuscleBuildingRecorderWatchTrueApp.swift # Entry Point
│   ├── ContentView.swift                # Main Watch View
│   ├── WorkoutManager.swift             # Watch-specific Session Logic
│   ├── BackgroundSensorRecorder.swift   # Motion Data Collection
│   └── WatchMotionStreamer.swift        # Real-time Data Streaming
│
├── Shared/                              # Shared Code (Swift Package)
│   └── Sources/                         # Shared Logic & Models
│
├── MuscleBuildingRecorderTests/         # iOS Unit Tests
├── MuscleBuildingRecorderUITests/       # iOS UI Tests
├── Package.swift                        # SPM Configuration for Shared Code
└── README.md, APP_STATUS.md, AGENTS.md  # Documentation
```

**Key Modules:**
-   **`SessionManager` (iOS) / `WorkoutManager` (watchOS):** Singleton classes managing the workout state (running, paused, resting), timer logic, and connectivity.
-   **`HeartRateService`:** Protocol-based abstraction for handling heart rate data from various sources (HK, BLE, AirPods).
-   **`WatchConnectivity`:** Utilizes `WCSession` for `sendMessage` (real-time), `updateApplicationContext` (state sync), and `transferFile` (bulk data).

## 3. Build & Run Instructions

**Environment:**
-   **IDE:** Xcode 16.0+
-   **iOS:** 17.0+
-   **watchOS:** 10.0+

**Build Commands:**
The project is primarily built using Xcode, but CLI commands can be used for CI/verification.

*   **Build iOS App:**
    ```bash
    xcodebuild -project MuscleBuildingRecorder.xcodeproj \
      -scheme MuscleBuildingRecorder \
      -destination 'platform=iOS Simulator,name=iPhone 15 Pro' \
      build
    ```

*   **Build Watch App:**
    ```bash
    xcodebuild -project MuscleBuildingRecorder.xcodeproj \
      -scheme "MuscleBuildingRecorderWatchTrue Watch App" \
      -destination 'platform=watchOS Simulator,name=Apple Watch Series 9 (45mm)' \
      build
    ```

*   **Run Tests:**
    ```bash
    xcodebuild test \
      -scheme MuscleBuildingRecorder \
      -destination 'platform=iOS Simulator,name=iPhone 15 Pro'
    ```

**Troubleshooting Build Errors:**
1.  **Clean Build Folder:** `Cmd + Shift + K` (or `xcodebuild clean ...`)
2.  **Derived Data:** Clear `~/Library/Developer/Xcode/DerivedData` if strange caching issues occur.
3.  **Package Reset:** `File -> Packages -> Reset Package Caches` if SPM dependencies fail.

## 4. Development Conventions

-   **Style:** Swift 5.9 standards. 4-space indentation.
-   **SwiftUI:** Prefer functional, declarative UI construction. Use `@StateObject`, `@ObservedObject`, and `@EnvironmentObject` appropriately.
-   **Concurrency:** Use Swift Concurrency (`async`/`await`) where possible, but be mindful of `Combine` pipelines existing in older service layers.
-   **Memory Management:** strictly use `[weak self]` in closures, especially for `Timer` and `WCSession` delegates, to prevent retain cycles.
-   **Error Handling:** Fail gracefully. Log errors to console but ensure the UI remains responsive.
-   **Testing:** Write XCTest cases for logic-heavy components (e.g., Timer calculation, Data parsing).

## 5. Critical Operational Rules for Agents

1.  **Preserve Existing Logic:** Do not rewrite complex logic (especially `HeartRateService` or `WatchConnectivity` handling) without fully understanding the existing flow. These areas have been patched multiple times (see `APP_STATUS.md` and `*_FIX_*.md` files).
2.  **Check `APP_STATUS.md`:** Before starting a task, briefly check this file to see the current state of known bugs and recent fixes.
3.  **Simulators:** When asked to "run" the app, clarify that you can build it or run tests, but you cannot launch a GUI simulator on the host machine.
4.  **File Creation:** When creating new files, ensure they are added to the correct target in the project structure (iOS vs watchOS vs Shared).
5.  **Sandboxing:** Respect the user's file system. Only modify files within the repository.

## 6. Known Issues & Context (from APP_STATUS.md)

-   **Watch Icon:** Missing icons error on TestFlight upload (Fixed by ensuring `CFBundleIconFiles` in Info.plist).
-   **Connectivity:** `isReachable` on Watch is flaky; use `updateApplicationContext` for reliable state sync.
-   **Memory:** High frequency sensor data (100Hz) must be buffered and batched (max 100 samples) to avoid OOM crashes on Watch.

Refer to `APP_STATUS.md` for the most up-to-date list of fixes and active issues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yok1012)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yok1012)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
