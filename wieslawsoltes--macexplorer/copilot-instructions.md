## macexplorer

> Keep all app-created UI in SwiftUI. Use small AppKit wrappers only for OS integration that lacks a SwiftUI surface. Keep filesystem/domain logic in ExplorerCore and off MainActor. Do not invent provider state or simulate native filesystem operations in the shipping app.

# Development conventions

Keep all app-created UI in SwiftUI. Use small AppKit wrappers only for OS integration that lacks a SwiftUI surface. Keep filesystem/domain logic in ExplorerCore and off MainActor. Do not invent provider state or simulate native filesystem operations in the shipping app.

Protect data: never overwrite without an explicit collision policy; never automatically purge replacement backups; never bypass OS permissions; no shell interpolation of paths; cancellation must not install incomplete copies; report partial completion truthfully. Add disposable-fixture tests for every transfer or recovery change.

Keep the capability matrix honest. Do not mark unsupported Windows-only shell contracts as implemented, confuse browser design interactions with native functionality, or claim validation from committed code alone.

Build using SwiftPM on macOS 14+ with Xcode. `swift test`, `bash scripts/package.sh`, and `bash scripts/verify-package.sh` cover separate layers. All release/Apple/Sparkle private keys stay out of the repository. Do not remove signed-feed verification to make an update test pass.

---
> Source: [wieslawsoltes/MacExplorer](https://github.com/wieslawsoltes/MacExplorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
