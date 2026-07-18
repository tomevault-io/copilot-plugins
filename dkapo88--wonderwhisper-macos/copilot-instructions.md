## 002-swift-style

> - **Indentation**: 2 spaces; ~100‑char line limit.

# Swift Style Guide

- **Indentation**: 2 spaces; ~100‑char line limit.
- **Naming**: types `PascalCase`; methods/vars `camelCase`; constants `static let`.
- **Files**: one primary type per file; filename matches type (e.g., `AudioTranscriber.swift`).
- **UI**: prefer SwiftUI with small, composable views and preview providers.
- **Formatting/Lint**: if configured, run `swiftformat .` and `swiftlint` before committing.

Favor clarity and explicitness. Avoid deep nesting beyond 2–3 levels; use guard clauses; handle errors explicitly.

---
> Source: [dkapo88/WonderWhisper-macOS](https://github.com/dkapo88/WonderWhisper-macOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
