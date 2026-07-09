## saypick

> macOS menu-bar app: **system-wide AI translation + inline rewrite**.

# Saypick — Agent Guide

macOS menu-bar app: **system-wide AI translation + inline rewrite**.

- **Read**: select text → shortcut / floating icon / auto → translation popup.
- **Write**: type in native language → shortcut → translated & replaced in place.

## Architecture

```
Shortcut / SelectionMonitor ─► SelectionCapture ─► TranslationService ─► provider
                                      │                                     │
                              PopupController / TextReplacer  ◄────── streamed result
```

- `Core/` — `SelectionCapture` (AX `kAXSelectedText` + synthetic-⌘C fallback, restores clipboard), `TextReplacer` (synthetic paste, undo-safe), `TriggerController` (orchestrates read/write), `SelectionMonitor` (global mouse-up → AX selection), `PopupPositioner`, `Keyboard`, `Pasteboard`, `AccessibilityPermission`, `LaunchAtLogin`.
- `Translation/` — `TranslationProvider` protocol; `OllamaProvider`, `OpenAIProvider` (SSE); `TranslationService` (routing + cache + style); `TranslationCache`; `OllamaModelResolver` (auto-pick installed model).
- `UI/` — `TranslationPopupView` + `PopupController`, `SelectionIconWindow`.
- `Config/` — `AppSettings` (single source of truth, UserDefaults-backed), `LanguageConfig`.
- `Services/` — `GlobalShortcutCenter` (Carbon, multi-hotkey), `UpdateChecker`, `SystemServiceProvider`.
- `Views/` — `MenuBarView`, `SettingsView/*` (General, Behavior, Backend, Language, Models, Shortcuts, Skip Apps, About).

## Conventions & gotchas

- **No sandbox** (`Saypick.entitlements` empty) — needs Accessibility + CGEvent posting. `LSUIElement = true` (menu-bar only).
- **Sign dev builds** (Apple Development team is configured) so the Accessibility grant persists across rebuilds; an unsigned/ad-hoc build’s grant won’t stick.
- Carbon hotkeys fire without Accessibility, but capture/replace are guarded by `AccessibilityPermission.isGranted`.
- Read direction: detected → `LanguageConfig.sourceLanguage` (native). Write: native → `targetLanguage`.
- Settings via `@AppStorage(AppSettings.Keys.*)`; behavior changes call `TriggerController.shared.applyEnabledState()`.

## Build

```bash
xcodebuild -scheme Saypick -configuration Debug build   # signed
./scripts/build-release.sh                                # notarized DMG (needs .env)
```

Backend: Ollama default `http://127.0.0.1:11434`; OpenAI-compatible base URL + key in settings.

Releases ship as notarized DMGs on **GitHub Releases at `everettjf/Saypick`** (where `UpdateChecker` also checks). Keep `README.md` free of the app's release version — link to `../../releases` and refer to the build output as `build/Saypick.dmg`, never a pinned `X.Y.Z`. Platform/dependency versions (macOS 13+, Swift 5.9+, Xcode 15+) are fine.

---
> Source: [everettjf/Saypick](https://github.com/everettjf/Saypick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
