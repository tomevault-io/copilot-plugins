## gingerpaw

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

FlowOSS is an OSS macOS push-to-talk dictation menu-bar app (Wispr Flow clone). Hold a hotkey → record mic → transcribe locally with WhisperKit → paste into the focused app. macOS 14+, Swift 6, strict concurrency `complete`.

## Commands

```sh
swift test                                   # all FlowKit tests (resolves & builds the whole graph)
swift test --filter pressStartsRecording     # single test by function name
swift build                                  # build the FlowOSSApp executable target
xcodegen generate                            # regenerate FlowOSS.xcodeproj from project.yml (after adding files/targets)
```

There is no SwiftPM-runnable app: `App/Sources` (the `@main` entry) requires the `.app` bundle (Info.plist, entitlements, TCC permissions) — build/run via Xcode or the generated project, not `swift run`. `swift build`/`swift test` exercise everything except the bundle. Tests use Swift Testing (`@Test`/`#expect`), not XCTest.

## Architecture

Two SwiftPM packages: the thin app shell at root (`App/Sources`, target `FlowOSSApp`) depends only on `AppCore` from the local `Packages/FlowKit`. All logic lives in FlowKit's nine library targets; the app shell is just `@main` + `AppDelegate` + the SwiftUI scenes.

**FlowKit dependency graph (leaf → root):**
- `Settings`, `Permissions`, `Audio`, `TextInsertion`, `Hotkeys` — independent leaves.
- `Transcription` → WhisperKit (from `argmax-oss-swift`). Defines `SpeechTranscriber` protocol + `WhisperKitTranscriber` actor.
- `Dictation` → Audio + Settings + TextInsertion + Transcription. The brain.
- `Overlay` → Dictation. The floating recording pill.
- `AppCore` → Dictation + Hotkeys + Overlay + Permissions + Settings. Composition + UI (menu bar, dashboard, settings).

**Wiring is centralized in two AppCore files** — start here for any cross-cutting change:
- `AppComposition.make()` constructs every service and injects concrete implementations into `DictationCoordinator` via protocols (`AudioRecording`, `SpeechTranscriber`, `TextInserter`). This seam is what lets tests inject stubs.
- `AppRuntime.shared.start()` is the runtime glue: wires `hotkeyMonitor.onPress/onRelease` → coordinator, and `coordinator.onStateChange` → overlay + status bar. Idempotent (`didStart` guard), called once from `AppDelegate`.

**`DictationCoordinator` is the state machine.** State flows `idle → recording → processing → inserting → (idle | copied | failed)`. All UI observes it via the `@Observable` macro and the `onStateChange` callback. `@MainActor`-isolated; the record→transcribe→insert chain runs in a detached `Task`. `copied`/`failed` auto-revert to `idle` after ~1.4s (`resetIdleSoon`). Add new states to the `DictationState` enum in `Dictation/DictationModels.swift` and update `canStartRecording`/`isBusy`.

### Conventions that matter

- **Protocol seams for testability:** every external dependency the coordinator touches is a protocol. New side-effecting services follow the same pattern so they can be stubbed (see `DictationCoordinatorTests`' `StubRecorder`/`StubTranscriber`/`StubInserter`).
- **Concurrency:** UI/coordination types are `@MainActor`. `WhisperKitTranscriber` is an `actor` (caches one WhisperKit pipeline per model id). `ClipboardTextInserter` is `@unchecked Sendable`. Respect these isolation boundaries — strict concurrency is on.
- **Three macOS TCC permissions** gate functionality, all surfaced through `PermissionCenter`: Microphone (AVFoundation), Input Monitoring (`CGPreflightListenEventAccess` — for the hotkey event tap), Accessibility (`AXIsProcessTrusted` — for synthetic ⌘V paste). Permission failures are silent at the OS level; the app polls/preflights rather than assuming grants.
- **Hotkey** = a global `CGEventTap` on `.flagsChanged` watching Right-Option (keycodes 58/61) and Fn (63 / `maskSecondaryFn`). It self-heals: re-arms on `tapDisabledBy*` and retries install every 1s until Input Monitoring is granted (`RightOptionHotkeyMonitor`).
- **Paste** = set clipboard, post synthetic ⌘V (`CGEvent` keycode 0x09), optionally restore the previous clipboard after a delay. Falls back to copy-only (`InsertionOutcome.copied`) when paste can't be sent or `autoPaste` is off.
- **Settings** persist to `UserDefaults` via `didSet`; inject a custom suite for tests (`UserDefaults(suiteName:)`).

## Project files

- `project.yml` — XcodeGen spec; the `.xcodeproj` is generated from it, so edit the yml, not the project. App bundle Info.plist keys (mic usage string, `LSUIElement`, etc.) live here.
- `Package.swift` (root) wires the app target to FlowKit; `Packages/FlowKit/Package.swift` declares the nine targets above.
- `dist/` holds prebuilt `.app`/`.dmg`/`.zip` artifacts — build output, not source.

---
> Source: [Vikrant-Khedkar/gingerpaw](https://github.com/Vikrant-Khedkar/gingerpaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
