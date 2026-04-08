## mneme

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mneme is a Swift 6 / iOS 26.1 app that turns free-form natural language text into structured actions (calendar events, reminders, expenses, meals/calories, journal entries, work sessions). It uses Apple Foundation Models (`SystemLanguageModel`) for on-device NLP — no cloud AI calls.

**Bundle ID:** `com.tekne.Mneme` | **License:** GPLv3

## Build & Test Commands

```bash
# Build
xcodebuild build -project Mneme.xcodeproj -scheme Mneme -destination 'platform=iOS Simulator,name=iPhone 16'

# Run all tests
xcodebuild test -project Mneme.xcodeproj -scheme Mneme -destination 'platform=iOS Simulator,name=iPhone 16'
```

No CocoaPods, Carthage, or SPM dependencies. Pure Xcode project.

LSP is configured via `buildServer.json` at repo root (requires `brew install xcode-build-server`).

## Architecture

**MVVM with Composition.** The main screen flow:

```
NotepadView → NotepadViewModel (thin coordinator)
                ├── LineStore        (dictionary-based notepad line state, persisted to JSON)
                ├── LineManager      (line business logic)
                ├── TagManager       (tag CRUD via CoreData)
                ├── EventKitManager  (calendar/reminder creation)
                ├── WorkSessionManager
                └── LocationManager
```

### Parsing Pipeline

1. User types in Notepad → `NotepadViewModel` debounces (300ms throttle, 800ms parse delay)
2. `VariableHandler` checks for variable expressions (`+salary`, `-rent`)
3. `FoundationModelsNLP` actor checks pattern-based shortcuts (calorie adjustments, currency nets)
4. `IntentClassificationService` uses `SystemLanguageModel` with `@Generable` struct to classify intent
5. `TranslationService` translates non-English input to English via LLM
6. Specialized parser services extract entities (`EventParserService`, `MealParserService`, etc.)
7. `NLPDateTimeSanitizer` validates/normalizes dates via regex
8. `HandlerFactory.handler(for:)` returns the appropriate `IntentHandler` → produces `[ParsingResultItem]`

### Supported Intents

`meal` | `expense` | `income` | `event` | `reminder` | `activity` | `work_start` | `work_end` | `journal` | `calorie_adjustment`

### Special Input Syntax

- `:` prefix → journal entry (mood picker)
- `@` prefix → location specifier
- `+variable` / `-variable` → variable arithmetic
- `+N kcal` / `-N kcal` → calorie adjustment

### Singleton Pattern

Services and stores use `.shared` singletons throughout: `NLPService.shared`, `PersistenceController.shared`, `NotepadEntryStore.shared`, `CurrencyService.shared`, `USDAService.shared`, etc.

### Core Data Model (`Mneme.xcdatamodeld`)

Key entities: `ParsedEntry` (primary record), `MealSource` (calorie sources), `CalorieLookupCache` (USDA cache), `DailyHealthStat` (HealthKit aggregates), `TagEntity`/`TagAssignment` (tagging), `Variable` → `VariableEntity`, `WorkSession`, `UserProfile` → `UserProfileEntity`.

Persistence uses `NSPersistentCloudKitContainer` with automatic CloudKit sync; falls back to local store on error.

### Concurrency

Swift 6 structured concurrency throughout:
- `FoundationModelsNLP` and `IntentClassificationService` are `actor` types
- All UI updates guarded by `@MainActor`
- NLP parsing runs on `Task.detached(priority: .userInitiated)`

## Key Directories

- `Mneme/Services/` — Business logic, NLP orchestration, external API integrations
- `Mneme/Handlers/` — Intent handler pattern (`HandlerProtocol.swift` defines `IntentHandler` + `HandlerFactory`)
- `Mneme/Stores/` — Observable CoreData-backed stores with `NSFetchedResultsController`
- `Mneme/Managers/` — Stateful coordinator objects (line state, EventKit, location)
- `Mneme/Helpers/` — Pure utility functions (date parsing, text normalization, number extraction)
- `Mneme/Charts/` — Analysis/charting subsystem (components, models, tabs, view models)
- `Mneme/Views/Notepad/` — Notepad-specific views including `LineTextView` (UIViewRepresentable)

## External APIs

- **FreeCurrencyAPI** — exchange rates. Key via `CURRENCY_API_KEY` env var (set in Xcode scheme)
- **USDA FDC** — calorie lookup. Key via `USDA_API_KEY` env var (falls back to `"DEMO_KEY"`)

## Entitlements

HealthKit (read-only), EventKit (Calendar + Reminders), CloudKit (`iCloud.com.tekne.Mneme`), Push Notifications, Location (when in use).

## Notable Behaviors

- `LineStore` clears `notepad_data.json` on first init each app session — notepad starts fresh each launch
- `NotepadEntryStore` uses `NSFetchedResultsController` for live CoreData updates
- `ContentView` adapts layout: `TabView` on iPhone, `NavigationSplitView` on iPad

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/teknetekne)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/teknetekne)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
