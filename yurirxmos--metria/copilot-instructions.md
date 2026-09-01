## metria

> - All repository text must be en-us, including Swift comments, UI strings, commit messages, and documentation; do not add Portuguese text.

# Agent Instructions

- All repository text must be en-us, including Swift comments, UI strings, commit messages, and documentation; do not add Portuguese text.
- The native macOS app is a Swift Package executable targeting macOS 13+, with the application entrypoint in `apps/macos-native/Sources/Metria/MetriaApp.swift`.
- The app is a menu-bar AppKit application whose dashboard, settings, and floating notch surface are SwiftUI views coordinated by `AppDelegate`.
- The floating surface is a "side notch": `NotchGeometry` anchors it flush against the right screen edge, hanging from just below the menu bar (`visibleFrame`, not the physical notch's `safeAreaInsets`); it stays edge-tight as a compact provider rail while idle and shows the hovered provider's card to the left.
- Run `swift build` from the repository root for the required verification.
- The native core usage and refresh logic lives in the `MetriaCore` library target (`apps/macos-native/Sources/MetriaCore`).
- The package manifest explicitly lists the source file and copies provider logos from `Assets/`; update `Package.swift` when adding or moving bundled assets.
- Usage credentials are read from macOS Keychain and local OpenCode/Codex files; do not commit credentials, generated `.build/` output, or local configuration.
- Provider selection, display mode, and notch opacity (applied only while expanded) are persisted through `UserDefaults`; preserve these keys when changing settings behavior.

---
> Source: [yurirxmos/metria](https://github.com/yurirxmos/metria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
