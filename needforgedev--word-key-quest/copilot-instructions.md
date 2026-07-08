## word-key-quest

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Word Key Quest is a Flutter vocabulary game for children ages 6-13, featuring 3,523 words (from Barron's 3500 master list) across 35 worlds x 10 levels x 10 words, plus bonus words. The app is designed as an offline-first adventure game with learning hidden inside the game loop. See `word_key_quest_flutter_product_spec.md` for full product spec and `plan.md` for the build plan.

## Common Commands

```bash
flutter run                    # Run the app (debug mode)
flutter build apk              # Build Android APK
flutter build ios              # Build iOS
flutter analyze                # Run static analysis (uses flutter_lints)
flutter test                   # Run all tests
flutter test test/widget_test.dart  # Run a single test file
flutter pub get                # Install dependencies
```

## Architecture

**Layered architecture** (planned, partially implemented):
- `presentation` - UI widgets and screens
- `application` - feature-scoped state controllers
- `domain` - entities and repository interfaces
- `data` - local persistence implementations

**Current state**: The presentation layer is fully built — all 20 screens match the design mockups (in `designs_word_key_quest/`) with proper styling, animations, and temporary network images from Google's CDN. Domain models, repositories, state management providers, and data layers are not yet implemented. See `plan.md` for the detailed execution plan.

### Current Project Structure

```
lib/
  main.dart                      # App entry point, ProviderScope wrapper
  app/
    router.dart                  # GoRouter with 21 flat routes (no nesting)
    theme/theme.dart             # AppTheme with color tokens + Google Fonts
  features/
    onboarding/                  # splash, parent_gate, child_profile_setup, theme_selection
    home/                        # home, daily_quest
    campaign/                    # world_map
    learn/                       # level_intro, learn_card, memory_key_focus, level_complete
    minigames/                   # image_match, meaning_tap, cue_recall, sentence_fix
    vault/                       # word_vault, word_detail
    rewards/                     # rewards_room
    parent/                      # parent_dashboard
    settings/                    # settings
    placeholder_screen.dart      # Generic placeholder widget
```

### Target Structure (not yet built)

```
lib/
  core/          # Shared services: audio/, storage/, analytics/, accessibility/, widgets/
  data/          # models/, repositories/, local_db/, seed/
```

These layers (`core/` and `data/`) are planned but not yet implemented.

### State Management & Routing

- **Riverpod** (`flutter_riverpod` ^3.3.1) for state management (ProviderScope set up, no providers defined yet)
- **GoRouter** (`go_router` ^17.1.0) for declarative routing with named routes (21 flat routes defined)
- **SharedPreferences** (`shared_preferences` ^2.5.5) for local persistence (dependency added, not yet used)
- **Google Fonts** (`google_fonts` ^8.0.2) for typography (Plus Jakarta Sans + Lexend)
- **Dart SDK**: ^3.11.0

### Navigation Flow

`Splash -> Parent Gate -> Profile Setup -> Theme Selection -> Home -> World Map -> Level Intro -> Learn Card -> Memory Key Focus -> Mini-games (Meaning Tap, Image Match, Cue Recall, Sentence Fix) -> Level Complete`

### Theme System

Uses Google Fonts (Plus Jakarta Sans for headings, Lexend for body). Three theme packs planned: Sky Heroes, Enchanted Kingdom, Explorer Quest. Currently only base color tokens are implemented in `AppTheme`.

### Mini-games

Four mini-game types sharing a common question/scoring contract (not yet implemented):
1. **Image Match** - Pick the correct image from 3 (best for ages 6-8)
2. **Meaning Tap** - Pick the correct `kidsMeaning` from 3 choices
3. **Cue Recall** - Pick the correct word from a memory key cue (most important for retention)
4. **Sentence Fix** - Choose the right word for a sentence blank

### Key Domain Concepts

- **Mastery levels 0-5**: 0=unseen, 1=introduced, 2=recognized, 3=recalled from cue, 4=used in context, 5=mastered
- **Spaced repetition intervals**: same day, 1d, 3d, 7d, 14d, 30d, 60d (presented as "quests"/"revisits", never as SRS UI)
- **Content manifests** (`words.json`, `worlds.json`, `levels.json`, `reward_catalog.json`) drive all content
- **Per-word fields** (from `barrons_3500_master copy - Sheet1.json`): `sourceRank`, `word`, `partOfSpeech`, `pronunciation`, `barronsMeaning`, `kidsMeaning`, `cueType`, `cue`, `memoryKey`, `nanoBananaPrompt`, `synonyms`, `antonyms`, `usage1Simple`, `usage2Standard`, `sourceInWorkbook`, `workbookSheet`, `promptProfile`, `imageProfile`
- **Per-level structure**: 10 new words + 2-4 review words + 1 boss challenge
- **Parent gate** guards access to settings/dashboard
- **Boss gate** every 5 levels in campaign progression
- **Adaptive difficulty**: fewer new words if accuracy drops, more review if cue recall is weak, more sentence play if recognition is strong

### Design Principles

- "Cue first" — the memory key is the main recall anchor, image supports it
- One clear action per screen to reduce cognitive overload
- Sessions are 5-10 minutes; every session ends with visible progress
- Themes are never gender-locked; any child can select any theme pack
- Two visual layers: soft stylized storybook game UI + realistic word images

### Word Data

The raw word dataset is in `barrons_3500_master copy - Sheet1.json` (3,523 words). This has not yet been processed into the content manifest format (`words.json`, `worlds.json`, `levels.json`) expected by the app. The `confusableWith` field mentioned in the spec is not present in the current dataset; additional fields `sourceInWorkbook`, `workbookSheet`, `promptProfile`, and `imageProfile` are present instead.

### What's NOT Built Yet

- No content manifest files (words.json, worlds.json, levels.json, reward_catalog.json) — raw data exists but not processed
- No domain models or entities (Dart classes)
- No local database (Hive/SQLite/etc.) or seed loader
- No repository layer (content, player, progress, review, reward, settings)
- No Riverpod providers or state management logic
- No mastery engine (0-5 state machine)
- No spaced repetition scheduler or daily quest generation
- No question engine (distractor generation, scoring)
- No adaptive difficulty controller
- No session manager (level flow orchestration)
- No audio system
- No real image assets (screens use temporary Google CDN URLs)
- No theme pack switching beyond base color tokens
- No tests
- All screens display hardcoded sample data — not yet wired to live state

---
> Source: [needforgedev/word_key_quest](https://github.com/needforgedev/word_key_quest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
