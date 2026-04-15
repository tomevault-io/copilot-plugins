## pule

> Cross-platform Flutter prayer list app. "Pule" = Hawaiian for prayer.

# Pule - Prayer List App

## Project Overview
Cross-platform Flutter prayer list app. "Pule" = Hawaiian for prayer.
Daily-reset checklist with tagging, share sheet, and cloud sync.

## Tech Stack
- **Flutter** with **Riverpod** state management
- **freezed** + **json_serializable** for immutable models
- **Storage**: iCloud (`NSUbiquitousKeyValueStore`) on iOS, Firebase Firestore on Android
- **Nix flake** dev environment with `just` task runner
- **Jujutsu** for version control

## Key Commands
- `just validate` — format check + analyze + test
- `just generate` — run freezed/json_serializable codegen
- `just run` — run the app
- `just deps` — get dependencies

## Architecture
- `lib/models/` — freezed data models (prayer_item, tag)
- `lib/repositories/` — abstract repo + platform-specific implementations
- `lib/providers/` — Riverpod providers
- `lib/screens/` — full-page screens
- `lib/widgets/` — reusable widget components
- `lib/services/` — share service, etc.

## Conventions
- Use Conventional Commits for all commit messages
- Run `just generate` after modifying freezed models
- Abstract repository pattern: never import platform-specific repos directly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kolohelios)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kolohelios)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
