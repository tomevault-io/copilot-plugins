## screencontext

> - Use the `build-macos-apps:swiftui-patterns` skill for user-facing macOS UI and interaction changes. Prefer native SwiftUI controls and standard macOS affordances before creating custom UI.

# ScreenContext Agent Guidance

## macOS UI

- Use the `build-macos-apps:swiftui-patterns` skill for user-facing macOS UI and interaction changes. Prefer native SwiftUI controls and standard macOS affordances before creating custom UI.
- Use the `build-macos-apps:appkit-interop` skill only when SwiftUI does not expose the required platform behavior. Keep any AppKit bridge narrow and leave SwiftUI as the source of truth.
- Preserve keyboard access, semantic labels, system symbols, adaptive colors, and standard action hierarchy when changing controls.

---
> Source: [marcusschiesser/screencontext](https://github.com/marcusschiesser/screencontext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
