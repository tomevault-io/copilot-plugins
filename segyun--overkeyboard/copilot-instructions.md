## overkeyboard

> `OverKeyboard` is a lightweight Swift Package that allows developers to present custom SwiftUI views above the iOS system keyboard.

# AGENTS.md - OverKeyboard

## Project Overview

`OverKeyboard` is a lightweight Swift Package that allows developers to present custom SwiftUI views above the iOS system keyboard.

- **Platforms**: iOS 13.0+
- **Swift Version**: Swift 6.0+
- **Key Capabilities**:
  - Dynamically observes iOS keyboard visibility and frame transitions.
  - Attaches and overlays custom SwiftUI content directly over the keyboard window (`UIRemoteKeyboardWindow` / `UITextEffectsWindow`) with automatic fallback to a custom `UIWindow` overlay.
  - Manages touch event hit-testing to ensure non-interactive areas pass touches through to underlying views.

---

## Common Commands

### Build

As an iOS-targeted Swift Package, build using `xcodebuild` or `swift build` targeted to an iOS simulator:

```bash
# Build with xcodebuild
xcodebuild -scheme OverKeyboard -destination "generic/platform=iOS Simulator"

# Or build with Swift PM CLI targeting iOS Simulator SDK
swift build -Xswiftc "-sdk" -Xswiftc "$(xcrun --sdk iphonesimulator --show-sdk-path)" -Xswiftc "-target" -Xswiftc "arm64-apple-ios17.0-simulator"
```

### Test

```bash
# Run tests with xcodebuild
xcodebuild test -scheme OverKeyboard -destination "platform=iOS Simulator,name=iPhone 16"
```

### Formatting & Linting (`swift-format`)

Code formatting and linting are handled via `swift-format`.

```bash
# Check / Lint code
swift-format lint -r Sources Examples

# Format code in-place
swift-format format -i -r Sources Examples
```

### Git Hooks

A pre-commit hook is provided in `.githooks/pre-commit` to automatically format staged `.swift` files and run lint checks before committing.

```bash
# Enable repository githooks
git config core.hooksPath .githooks
```

---

## Project Architecture & Directory Structure

```text
OverKeyboard/
|-- Package.swift                  # Swift package manifest (Swift 6.0, iOS 13+)
|-- .swift-format                  # swift-format configuration file
|-- .githooks/
|   \-- pre-commit                 # Git pre-commit hook for swift-format
|-- Sources/
|   \-- OverKeyboard/
|       |-- OverKeyboard.swift             # Public SwiftUI View / modifier API for overlay presentation
|       |-- OverKeyboardPresenter.swift    # @MainActor singleton orchestrating view attachment & lifecycle
|       |-- KeyboardWindowObserver.swift   # @MainActor observer tracking keyboard events via async notifications
|       |-- OverKeyboardHostView.swift     # Custom UIView subclass managing hit-test & touch pass-through
|       \-- OverKeyboardWindow.swift       # Custom UIWindow subclass for overlay presentation
|-- Examples/
|   \-- ChatGPTModelPicker/        # Showcase example app integrating OverKeyboard
|       |-- ChatGPTModelPicker.xcodeproj
|       \-- ChatGPTModelPicker/
\-- Tests/
    \-- OverKeyboardTests/                 # Unit and integration test suites
```

### Component Roles

- `OverKeyboard.swift`: Provides the declarative SwiftUI wrapper (`OverKeyboard<Content>`) that binds to an `isPresented` state.
- `OverKeyboardPresenter.swift`: Manages the presentation lifecycle, coordinating `UIHostingController` instances and attaching/detaching views to the keyboard window or fallback window.
- `KeyboardWindowObserver.swift`: Listens to `UIWindow.didBecomeVisibleNotification`, `UIResponder.keyboardWillChangeFrameNotification`, and `UIResponder.keyboardWillHideNotification` to keep track of keyboard state.
- `OverKeyboardHostView.swift` & `OverKeyboardWindow.swift`: Implement custom `hitTest(_:with:)` logic to allow transparent regions to pass touches through without blocking interactions with underlying UI elements.
- `Examples/ChatGPTModelPicker`: Demonstration app showcasing a ChatGPT-style floating model picker over the system keyboard with custom glass effects and spring animations.

---

## Code Conventions & Guidelines

- **Swift 6 Concurrency**:
  - Code must adhere to strict concurrency safety and Swift 6 standards (`.enableUpcomingFeature("ApproachableConcurrency")`).
  - UI-coordinating classes (`OverKeyboardPresenter`, `KeyboardWindowObserver`) must be isolated to `@MainActor`.
  - Prefer modern structured concurrency (e.g., `Task`, `for await` notification sequences via `NotificationCenter.default.notifications(named:)`)` over legacy selector-based observers or completion handlers.
- **SwiftUI & UIKit Interoperability**:
  - Maintain clear separation between public SwiftUI declarative APIs and internal UIKit window hierarchy manipulations.
- **Formatting**:
  - Always ensure changes comply with `swift-format lint -r Sources Examples`.

---

## Git Workflow & Commit Conventions

- Follow Conventional Commits format (`type: concise description in lowercase/imperative`).
- **Commit Types**:
  - `feat:` New feature or capability
  - `fix:` Bug fix
  - `refactor:` Code change that neither fixes a bug nor adds a feature
  - `docs:` Documentation updates
  - `test:` Adding or updating tests
  - `chore:` Maintenance tasks, dependency updates, or initial configuration (e.g., `chore: initial commit`)
- **Branch Naming**:
  - Use `codex/` or `feature/`, `fix/` prefixes (e.g., `codex/add-keyboard-dismiss-gesture`, `fix/ios16-hittest`).

---
> Source: [Segyun/OverKeyboard](https://github.com/Segyun/OverKeyboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
