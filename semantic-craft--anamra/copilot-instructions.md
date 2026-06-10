## anamra

> Repo-level guidance for coding agents (OpenCode, Claude Code, Codex, and others).

# AGENTS.md

Repo-level guidance for coding agents (OpenCode, Claude Code, Codex, and others).

## Project

Anamra (安记) — native macOS menu-bar app: capture → AI classify → confirmable tasks + semantic memory.
SwiftUI + AppKit + Core Swift Package + SwiftData. macOS 14+.
Bundle id: `com.xianwei.anamra`. GitHub: `semantic-craft/anamra`.

## Build & verify

Prerequisites: `brew install xcodegen`.

```bash
scripts/verify.sh              # Core tests → macOS build (full gate)
(cd Core && swift test)        # Core unit tests only
scripts/build.sh               # macOS build only (xcodegen + xcodebuild)
scripts/release.sh <mode>      # key|check|archive|package|github|all
```

## Architecture

```
project.yml          ← XcodeGen: edit this, NEVER .xcodeproj (gitignored)
Core/                ← Local Swift Package: pure Swift, no AppKit/SwiftUI
  Package.swift      ← swift-tools-version 6.0, Swift 5 language mode
  Sources/Core/      ← Config/, Models/, Services/, Memory/
  Tests/CoreTests/   ← all run via `swift test`
macOS/
  Sources/           ← App layer: SwiftUI views + AppKit helpers
  Resources/         ← sound assets
  Info.plist         ← LSUIElement=true (menu-bar only, no Dock icon)
scripts/             ← build.sh, verify.sh, release.sh
```

Key boundaries:
- **Core has zero Apple UI framework imports.** All AppKit/SwiftUI/SwiftData lives under `macOS/Sources/`.
- **Edit `project.yml`, then run `xcodegen generate`** (or `scripts/build.sh` which does it for you).
- **Swift 5 language mode everywhere** (`Package.swift`: `.swiftLanguageMode(.v5)`, `project.yml`: `SWIFT_VERSION: "5.0"`), despite swift-tools-version 6.0.

## Domain model

Read `CONTEXT.md` and relevant `docs/adr/` before working in an area. Key terms:
- `TaskCandidate` — confirmable task; canonical data in on-device SwiftData.
- `WritePlan` / `WriteReceipt` — plan + evidence for one-way mirror to Apple apps.
- Apple Reminders/Calendar/Notes are **optional one-way sync exits**. Only completion state is read back (one-way, completion-only). Never two-way merge.
- `Clarification` — multi-turn conversation refining one TaskCandidate; transcript is session-only.
- AI output must become a draft before writing to external systems.

## Design system

- `macOS/Sources/DesignTokens.swift` — `DS` (popover/compact) and `DS.Window` (main window) tokens.
- Apple HIG only: semantic colors, system materials, SF Symbols monoline, 8px grid. No hardcoded dark/light pairs.
- See `docs/DESIGN.md`.

## Conventions

- SwiftUI for UI; AppKit only for macOS-specific needs (windows, panels, capture, menu-bar).
- Root-level `test_*.swift` files are scratch experiments — ignore them.
- `ANAMRA_SEED=1` in DEBUG: in-memory SwiftData (no persistence), sample data injected, reminder polling + notifications suppressed. Also `ANAMRA_PREVIEW=panel|main|history|settings` for debug window selection.
- API keys are user-provided (BYOK), stored in Keychain — never hardcode or log keys.

## Agent skills

- Issue tracker: `semantic-craft/anamra` GitHub Issues via `gh` CLI. See `docs/agents/issue-tracker.md`.
- Triage labels: canonical roles `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.
- Domain docs config: single-context layout; see `docs/agents/domain.md`.

---
> Source: [semantic-craft/anamra](https://github.com/semantic-craft/anamra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
