## momentum

> Momentum is an iOS task manager application built with Swift and SwiftUI. It leverages SwiftData for persistence and is structured as a Swift Package with modular components for core logic, authentication, UI, and features. The app includes features like task management, calendar view, and statistics, with biometric authentication for security.

# Project Overview

Momentum is an iOS task manager application built with Swift and SwiftUI. It leverages SwiftData for persistence and is structured as a Swift Package with modular components for core logic, authentication, UI, and features. The app includes features like task management, calendar view, and statistics, with biometric authentication for security.

# Building and Running

The project can be built and run using Xcode.

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/jiaruh/Momentum.git
    ```
2.  **Open in Xcode:**
    Open `Momentum.xcodeproj` in Xcode.
3.  **Build and Run:**
    Select a target (e.g., an iOS Simulator or a connected device) and build and run the project from Xcode.

Alternatively, for command-line building (as seen in CI):

```sh
xcodebuild clean build \
  -project Momentum.xcodeproj \
  -target Momentum \
  -destination 'generic/platform=iOS Simulator' \
  CODE_SIGN_IDENTITY="" \
  CODE_SIGNING_REQUIRED=NO
```

# Development Conventions

*   **Modular Architecture:** The project is organized into distinct Swift Package modules (`MomentumCore`, `MomentumAuthentication`, `MomentumUI`, `MomentumFeatures`), promoting code reusability and separation of concerns.
*   **SwiftUI:** The user interface is declaratively built using Apple's SwiftUI framework.
*   **SwiftData:** Data persistence and management are handled by SwiftData.
*   **Strict Concurrency:** The project utilizes Swift's concurrency features with the experimental "StrictConcurrency" setting enabled in `Package.swift`, aiming for safer concurrent programming.
*   **Authentication:** The application incorporates an `AuthenticationService` to manage user authentication, including support for biometric methods.
*   **Testing:** Dedicated test targets are provided for each module (e.g., `MomentumCoreTests`, `MomentumAuthenticationTests`), indicating a commitment to testing and code quality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jiaruh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
