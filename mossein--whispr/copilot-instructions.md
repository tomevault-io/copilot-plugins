## whispr

> xcodebuild -project Whispr.xcodeproj -scheme Whispr -configuration Release -derivedDataPath .build/xcode build

# Whispr - Development Notes

## Build

```bash
xcodegen generate
xcodebuild -project Whispr.xcodeproj -scheme Whispr -configuration Release -derivedDataPath .build/xcode build
```

Sign with developer cert after building:
```bash
codesign -fs "Apple Development: mooshe_ashpaz@yahoo.com (482F852EN2)" --deep .build/xcode/Build/Products/Release/Whispr.app
```

XcodeGen wipes `Whispr.entitlements` on every `generate` - entitlements are defined in `project.yml` under `entitlements.properties` instead.

## Architecture

- **AppKit for UI chrome** (NSStatusBar, NSMenu, NSWindow) - SwiftUI MenuBarExtra was unreliable
- **SwiftUI for content views** inside NSHostingController - but never use `@State` in views embedded via NSHostingView/NSHostingController. All mutable state must live on the NSViewController and views re-render by swapping the hosted view.
- **AppCoordinator** is the central orchestrator. Everything flows through it.
- **NSEvent.addGlobalMonitorForEvents** for key detection (not CGEventTap - that had permission issues)

## Permissions

- Accessibility permission is tied to code signature TeamIdentifier. Always sign with the developer cert, not ad-hoc (`-`), or accessibility breaks on every rebuild.
- Paste uses CGEvent to simulate Cmd+V. This requires accessibility permission.
- `tccutil reset Accessibility com.whispr.app` to reset permissions if things get stuck.

## WhisperKit

- `transcribe(audioArray:)` hangs with VAD chunking enabled. Always pass `DecodingOptions(chunkingStrategy: .none)`.
- First transcription after launch is slow (~30s) due to CoreML model compilation. Subsequent calls are <200ms.
- Model downloads to `~/Documents/huggingface/models/argmaxinc/whisperkit-coreml/`.

## Homebrew

Tap repo: `github.com/mossein/homebrew-whispr`. Update `Casks/whispr.rb` with new version and sha256 on each release.

```bash
shasum -a 256 dist/Whispr-macOS.zip
```

## Release checklist

1. Build and sign
2. `cp -R .build/xcode/Build/Products/Release/Whispr.app dist/`
3. `cd dist && zip -r Whispr-macOS.zip Whispr.app`
4. `shasum -a 256 dist/Whispr-macOS.zip`
5. `gh release create vX.Y.Z dist/Whispr-macOS.zip --title "..." --notes "..."`
6. Update `homebrew-whispr/Casks/whispr.rb` with new version + sha256

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mossein)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mossein)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
