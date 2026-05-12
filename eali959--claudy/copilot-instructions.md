## claudy

> Claud-y is a macOS desktop companion app. A small, round, animated orange creature lives in a floating transparent window on the user's screen. It reacts to your work, tracks your focus sessions, and can chat using your choice of AI provider. Think: Clippy but adorable, intelligent, and hilarious.

# Claud-y — macOS Floating AI Companion

## Project Overview
Claud-y is a macOS desktop companion app. A small, round, animated orange creature lives in a floating transparent window on the user's screen. It reacts to your work, tracks your focus sessions, and can chat using your choice of AI provider. Think: Clippy but adorable, intelligent, and hilarious.

## Platform & Stack
- Platform: macOS 15+
- Language: Swift 6.0 strict concurrency
- UI: SwiftUI only (no UIKit, no AppKit unless absolutely necessary for floating window)
- Architecture: MVVM with @Observable
- Animation: Pure SwiftUI (custom-drawn character in `ClaudyCharacterView.swift`)
- AI: Claude (Anthropic) · ChatGPT (OpenAI) · Gemini (Google) — all via REST, no SDK dependency
- Package Manager: Swift Package Manager (no third-party dependencies)

## Key Architecture Rules
- Character lives in a transparent, draggable `NSPanel` with `isFloatingPanel = true`
- Always-on-top via `.windowLevel(.floating)` and `windowManagerRole(.associated)`
- Quick-chat panel slides in from the character using SwiftUI sheet/popover
- Menu bar `NSStatusItem` controls personality, mode, settings, quit
- All API calls are async/await via actor `ClaudeAPIService` — routes by `APIProvider.selected`
- Use `@Observable` not `ObservableObject`
- Use `NavigationStack` not deprecated `NavigationView`
- Extract SwiftUI views when they exceed 100 lines

## Coding Standards
- Swift 6 strict concurrency throughout — all new managers are `@MainActor`
- Prefer value types (structs) over classes where possible
- Use `guard` for early exits
- All async operations via `async/await`
- No `print()` for logging — use `Logger` from `OSLog`
- SF Symbols for all iconography
- Aim for Apple Human Interface Guidelines compliance

## Claud-y Character Specs
- Round orange creature (terra cotta / `#C15F3C` palette)
- Big expressive eyes, nub arms, floats and bobs
- Animation states: `idle`, `thinking`, `talking`, `celebrating`, `confused`, `sleeping`, `surprised`, `dancing`, `headbanging`, `vibing`
- All animations are pure SwiftUI — no external dependencies
- Draggable anywhere on screen — position persisted to `UserDefaults["CharacterWindowOrigin"]`

## AI Provider Architecture
All API calls go through `ClaudeAPIService` (actor). Provider is selected via `APIProvider.selected` (reads `UserDefaults["SelectedProvider"]`).

| Provider | Auth | Streaming | Fast model | Default model | Smart model |
|----------|------|-----------|------------|---------------|-------------|
| `.claude` | `x-api-key` header | SSE `content_block_delta` | `claude-3-5-haiku-20241022` | `claude-haiku-4-5-20251001` | `claude-sonnet-4-6` |
| `.openai` | `Authorization: Bearer` | SSE `choices[0].delta.content` | `gpt-4o-mini` | `gpt-4o-mini` | `gpt-4o` |
| `.gemini` | `?key=` URL param | SSE `candidates[0].content.parts[0].text` | `gemini-2.0-flash` | `gemini-2.0-flash` | `gemini-1.5-pro` |

- API keys stored in macOS Keychain via `KeychainService` — per-provider accounts (`claude-api-key`, `openai-api-key`, `gemini-api-key`)
- `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` — never synced to iCloud, never in UserDefaults
- Rate-limit unprompted commentary to max 1 per 60 seconds
- Conversation history in memory only — not persisted

## Personality × Mode System
Two orthogonal axes that stack in the system prompt: `[base] + [personality block] + [mode block]`

**Personalities** (7): Companion, Chatty, HypeCoach, Director, Mate, Listener, Custom
- Managed by `PersonalityManager` (`@Observable` singleton)
- `currentMode: PersonalityMode` drives system prompt base

**Behaviour Modes** (6): Normal, Study, Dev, Work, Dance, BrainRot
- Managed by `BehaviorModeManager` (`@MainActor` class, weak ref to `CharacterViewModel`)
- `currentMode: BehaviorMode` drives ambient cooldown multiplier and mode prompt block
- Each mode has `onAppSwitch(trigger:)` and `onBuildComplete(success:)` hooks
- Ambient cooldown multipliers: Normal 1×, Study 3×, Dev 0.75×, Work 1.8×, Dance 0.5×, BrainRot 0.5×

## Background Manager Architecture (v2.0)
Eight `@MainActor` singleton managers wired in `CharacterViewModel.init()`:

| Manager | Purpose | Key behaviour |
|---------|---------|---------------|
| `GlobalHotkeyManager` | ⌘⇧Space global hotkey | `NSEvent` global monitor, posts `.claudyToggleChat` |
| `FocusModeMonitor` | macOS Focus/DND sync | `NSDistributedNotificationCenter` for DND + screen sleep/lock |
| `BreakNudgeManager` | Screen-time break nudges | 90/120/180 min thresholds, personality+mode aware |
| `FocusStatsManager` | Pomodoro + focus stats | Daily rollover, streak tracking, `UserDefaults["FocusStats"]` |
| `QuickActionManager` | Contextual quick-action buttons | App-switch → capsule prompt above character, 8s auto-dismiss |
| `ScratchpadManager` | Persistent scratchpad notes | CRUD + pin, `UserDefaults["ScratchpadNotes"]` |
| `MoodCheckInManager` | Mood check-ins every ~2h | 30-min support mode on struggle signal |
| `DailyWrapUpManager` | 6pm daily wrap-up | Fires once/day if ≥1 Pomodoro, personality-flavoured |

Notification names (defined in `CharacterViewModel`):
- `.claudyToggleChat` — toggle chat panel
- `.claudyQuickActionFired` — pre-fill `ChatViewModel.inputText` with prompt

## App & Browser Awareness
`AppContextMonitor` maps frontmost app bundle IDs → `ReactionTrigger` enum cases.
`ReactionLibraryService` maps trigger cases → JSON keys in `ReactionLibrary.json`.

**Browsers detected**: Chrome, Safari, Edge, Firefox, Brave, Opera, DuckDuckGo, Helium, Arc

**App categories**: Xcode, Cursor, VS Code, Windsurf, Antigravity, Figma, Slack, Zoom, Teams, Notion, Obsidian, GitHub Desktop, Linear, Raycast, Postman, Insomnia, full Microsoft Office suite, full Apple productivity suite, and more.

`BehaviorModeManager.onAppSwitch(trigger:)` is called on every app switch — each mode can react contextually (e.g. Work mode reacts to Zoom/Teams/Outlook differently than Study mode reacts to browsers).

`QuickActionManager.appDidActivate(bundleID:)` is also called — surfaces a pre-written contextual prompt for 8 seconds.

## Files to Always Respect
- `ClaudyCharacterView.swift` — animated character drawing — never refactor structure without asking
- `PersonalityManager.swift` — personality mode state and prompt injection
- `ClaudeAPIService.swift` — **all API calls live here, nowhere else** — routes by provider
- `KeychainService.swift` — all Keychain access + `APIProvider` enum definition
- `BehaviorModeManager.swift` — all behaviour mode logic, never split across files
- `ReactionLibrary.json` — reaction strings, keys must match `ReactionTrigger` raw values exactly

## Key UserDefaults Keys
| Key | Type | Notes |
|-----|------|-------|
| `SelectedProvider` | String | `"claude"` / `"openai"` / `"gemini"` |
| `SelectedModel` | String | Provider-specific model ID |
| `UseComplexModel` | Bool | Use smart model for complex tasks |
| `GlobalHotkeyEnabled` | Bool | ⌘⇧Space toggle |
| `ChattinessLevel` | Int | 1–5 ambient bubble frequency |
| `FocusStats` | Data | JSON `StoredStats` — Pomodoros, focus seconds, streak |
| `ScratchpadNotes` | Data | JSON array of `ScratchpadNote` |
| `CharacterWindowOrigin` | [Double] | Last panel position |
| `CharacterSizePreset` | String | `"small"` / `"medium"` / `"large"` |

## Build & Run
Use XcodeBuildMCP for all build/test/run operations:
- Build: `mcp__xcodebuildmcp__build_sim_name_proj`
- Target: macOS (not simulator)
- Clean on error before retry
- No external dependencies — `swift package resolve` not needed

---
> Source: [eali959/claudy](https://github.com/eali959/claudy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
