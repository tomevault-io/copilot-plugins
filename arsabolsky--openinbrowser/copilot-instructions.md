## openinbrowser

> This document provides architectural overview, development standards, and operational guidelines for the OpenInBrowser project.

# Gemini Instructional Context - OpenInBrowser

This document provides architectural overview, development standards, and operational guidelines for the OpenInBrowser project.

## 🚀 Project Overview

**OpenInBrowser** is a macOS utility designed to function as the system's default web browser. Its primary responsibility is not to render web pages, but to intercept link clicks (HTTP/HTTPS) and provide a user interface for selecting which installed browser and specific profile (e.g., Chrome "Work", Safari "Personal") should handle the request.

### Core Technologies
- **Language:** Swift 5+
- **Frameworks:** SwiftUI (UI), AppKit (System integration)
- **Scripting:** AppleScript (via `NSAppleScript`) for Safari profile automation.
- **Data Access:** SQLite (via `sqlite3` CLI) for reading Safari profile metadata.
- **Process Management:** `Process` (NSTask) for launching browser binaries directly.

### Architecture
The project follows a lightweight MVVM-like structure:
- **OpenInBrowserApp.swift:** Entry point. Uses `.onOpenURL` to intercept system-wide link clicks.
- **AppViewModel.swift:** Central state manager. Maintains the list of discovered browsers and handles the launch lifecycle.
- **DiscoveryEngine.swift:** Responsible for scanning the system for browsers (`NSWorkspace`) and parsing their profile configurations (Chromium `Local State`, Firefox `profiles.ini`, Safari `SafariTabs.db`).
- **ExecutionEngine.swift:** Handles the hand-off to the target browser. 
    - **Chromium/Firefox:** Launches the internal binary directly with profile-specific flags.
    - **Safari:** Uses UI Scripting to navigate the **File > New Window** submenu.
- **Models.swift:** Defines `Browser` and `BrowserProfile` structures.

## 🛠 Building and Running

### Prerequisites
- **Xcode:** Version 15+ (targeted for macOS 14/15 SDKs).
- **macOS:** 14.0+ (requires modern `NSWorkspace` and Safari Profile APIs).

### Build Configuration
- **App Sandbox:** MUST be disabled in **Signing & Capabilities**. The app requires access to other application binaries and system-level databases (Safari).
- **Info.plist:** Configured with `LSUIElement = true` to run as a background utility (no Dock icon).
- **URL Schemes:** Handled via `CFBundleURLTypes` for `http` and `https`.

### Running & Testing
1. Build and Run via Xcode (**Cmd + R**).
2. Set **OpenInBrowser** as the default browser in **System Settings > Desktop & Dock**.
3. Grant **Accessibility** and **Automation** (for Safari) permissions in **System Settings > Privacy & Security**.
4. Click a link in any non-browser app (e.g., Notes) to trigger the selection UI.

## 📝 Development Conventions

### Coding Style
- **Naming:** Follow standard Swift API Design Guidelines.
- **Logging:** Maintain extensive `print` statements for system-level discovery and execution steps to aid in debugging complex macOS security interactions.
- **UI:** Prefer declarative SwiftUI. The main window is transient and should hide immediately after an action (`NSApp.hide(nil)`).

### Testing Practices
- **Unit Tests:** Located in `OpenInBrowserTests`. Focus on parsing logic (JSON, INI, SQLite outputs).
- **Manual Verification:** Since the app relies heavily on inter-app communication, manual testing with various browsers (Chrome, Firefox, Safari, Helium) is essential.

### Known Limitations
- **Safari Automation:** Heavily dependent on AppleScript UI paths, which may change between macOS versions. Currently optimized for macOS Sequoia.
- **Full Disk Access:** Discovery of profile names often requires FDA to read browser databases in `~/Library/Containers`.

## ✅ TODO / Future Scope
- Implement a **Rules Engine** for domain-based auto-routing.
- Add a **Menu Bar only** mode.
- Improve **Firefox** direct binary launch reliability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arsabolsky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/arsabolsky)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
