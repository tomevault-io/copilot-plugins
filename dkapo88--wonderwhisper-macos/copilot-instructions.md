## 003-build-and-test

> Build, Test, and Development commands for WonderWhisper

# Build, Test, and Development

- **Open in Xcode**:
  - `open "WonderWhisper.xcodeproj"` (use `.xcworkspace` if present).
- **Build (Debug)**:
  - ```bash
    xcodebuild -project "WonderWhisper.xcodeproj" -scheme "WonderWhisper" -configuration Debug build
    ```
- **Test (macOS)**:
  - ```bash
    xcodebuild -project "WonderWhisper.xcodeproj" -scheme "WonderWhisper" -destination 'platform=macOS' test
    ```
- **Run locally**:
  - Use Xcode, or `./Scripts/build-and-run.sh` for a script-built Debug app.

Prefer absolute paths in commands within this workspace.

---
> Source: [dkapo88/WonderWhisper-macOS](https://github.com/dkapo88/WonderWhisper-macOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
