## habittracker

> Single-module Jetpack Compose app · `com.plcoding.habittracker`

# Habit Tracker

Single-module Jetpack Compose app · `com.plcoding.habittracker`

## Resources

| What | Where |
|------|-------|
| Requirements | `specs/habit-tracker-design.md` — read when you need project rules or behavior details |
| UI mockups | `specs/Habit Tracker — Design System & Screens.html` — read only the section for the screen you're building |
| Fonts | `specs/Fonts/` — Inter and Manrope variable fonts |

Do **not** inline requirements or design details into code comments. Always lazy-load from the specs when needed.

## Mandatory Skill Loading

Before working on a layer, **always load the corresponding skill first**.

| Layer | Skill to load |
|-------|---------------|
| Presentation / MVI | `android-presentation-mvi` |
| Compose screen architecture | `android-compose-architecture` |
| Compose UI components | `android-compose-components` |
| Data layer (repos, data sources) | `android-data-layer` |
| Dependency injection (Koin) | `android-di-koin` |
| Navigation | `android-navigation` |
| Error handling / Result types | `android-error-handling` |
| Background work / WorkManager | `android-background` |
| Coroutines | `kotlin-coroutines` |
| Flows | `kotlin-flows` |
| Version catalog / Gradle | `android-version-catalog` |
| Testing | `android-testing` |

## Rules

- **Single module, layered packages.** Use the same package structure as the `android-module-structure` skill (core, feature, etc.) but as packages within `:app`, not separate modules.
- **No tests unless asked.** Do not write tests unless the user explicitly requests them.
- **Lazy mockup loading.** When building UI, read the HTML mockup file for that specific screen only — do not read the entire file upfront.
- **Git hygiene.** `git add` every new file immediately after creating it. Create meaningful, modular commits at logical checkpoints — don't batch everything into one giant commit.

---
> Source: [philipplackner/HabitTracker](https://github.com/philipplackner/HabitTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
