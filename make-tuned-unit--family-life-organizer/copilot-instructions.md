## family-life-organizer

> Native iOS companion app for the Family Life Organizer web app. Built for Jesse and Melissa to manage their household from their iPhones. Syncs with the existing Express/SQLite backend.

# FamilyLife iOS — Claude Agent Guide

## Project Context

Native iOS companion app for the Family Life Organizer web app. Built for Jesse and Melissa to manage their household from their iPhones. Syncs with the existing Express/SQLite backend.

The web app lives at `../family-life-organizer/` — reference it for API contracts, data models, and feature parity.

## Tech Stack

- SwiftUI + SwiftData, iOS 18+, Xcode 16+
- MVVM with `@Observable`
- Swift Concurrency (async/await)
- Bundle ID: `com.atlasatlantic.familylife`

## Architecture

```
FamilyLife/
├── App/                    # FamilyLifeApp.swift, ContentView, app config
├── Views/
│   ├── Home/               # Dashboard overview, summary cards
│   ├── Calendar/           # Monthly calendar, appointment detail
│   ├── Pantry/             # Inventory browser, expiry indicators
│   ├── Expenses/           # Budget progress, receipt scanner
│   └── Cook/               # AI recipe suggestions, pantry query
├── Models/                 # SwiftData models matching family.db schema
├── Services/               # APIService, SyncEngine, AuthService
└── Resources/              # Assets.xcassets, colors, app icon
```

### Key Patterns

- **Views** are SwiftUI structs, thin on logic
- **ViewModels** are `@Observable` classes, one per feature area
- **Models** mirror the SQLite schema (tasks, groceries, appointments, receipts, pantry, budget_categories)
- **APIService** is a single async client wrapping all Express API endpoints
- **SyncEngine** manages local SwiftData cache ↔ remote API state

## API Base URL

Configurable — defaults to the local network address where the Express app runs. The web app exposes 18 REST endpoints (see `docs/PRD.md` for the full list).

## Instructions for Claude Agent

1. **Use SwiftUI previews** — every view should have a `#Preview` with mock data so iteration is fast
2. **Commit after each feature** — one feature = one commit. Keep commits atomic and descriptive
3. **Prefer incremental builds** — get a minimal version working first, then layer on polish
4. **Match the web app's data model** — the SwiftData models should map cleanly to the SQLite schema in `../family-life-organizer/schema.sql`
5. **Keep views simple** — extract reusable components (cards, badges, progress bars) into a shared Components/ folder when patterns emerge
6. **Use SF Symbols** — no custom icons unless absolutely necessary
7. **Handle errors gracefully** — show inline error states, not alerts, for API failures
8. **Test on iPhone first** — iPad layout is secondary
9. **No third-party dependencies** unless strictly necessary — prefer Apple frameworks
10. **Reference the PRD** at `docs/PRD.md` for feature requirements and design guidelines

---
> Source: [make-tuned-unit/family-life-organizer](https://github.com/make-tuned-unit/family-life-organizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
