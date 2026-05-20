## one

> Kotlin Android + Wear OS fasting tracker. Jetpack Compose UI, Koin DI, Room DB, DataStore, Kotlin Flows. Min SDK 31.

# ONE Fasting Tracker

Kotlin Android + Wear OS fasting tracker. Jetpack Compose UI, Koin DI, Room DB, DataStore, Kotlin Flows. Min SDK 31.

## Before You Start

1. Read `@docs/ARCHITECTURE.md` for the full project structure
2. Ask: **does this change need to sync between phone and watch?**

## Do Not

- **Break sync** between phone and watch — all fasting state changes MUST propagate via Wearable Data Layer
- **Forget UI component updates** after data changes — update ALL of: widgets (`WidgetUpdateManager`), complications (`ComplicationUpdateManager`), tiles (`TileService.requestUpdate()`), ongoing activities (`OngoingActivityService`)
- **Omit timestamps** in sync updates — required for conflict resolution
- **Create local-only state** that should be synced — use coroutines and Flows everywhere, never blocking calls
- **Add dependencies** without going through `gradle/libs.versions.toml` first
- **Skip Koin registration** for new ViewModels or repositories
- **Filter incorrectly** in listener services — always filter local events to prevent sync loops

## Architecture Rules

### Dependency flow

```
app ──→ features:dashboard:app  ──→ core
    ──→ features:profile        ──→ core
    ──→ features:settings       ──→ core
wear ──→ features:dashboard:wear ──→ core
     ──→ core
```

### Modules

| Module                     | Purpose                                                                  |
| -------------------------- | ------------------------------------------------------------------------ |
| `:app`                     | Phone/tablet app — Compose UI, Glance widgets, WorkManager notifications |
| `:wear`                    | Wear OS app — Wear Compose, complications, tiles, ongoing activities     |
| `:features:dashboard:app`  | Phone dashboard / today screen feature                                   |
| `:features:dashboard:wear` | Wear OS dashboard / today screen feature                                 |
| `:features:profile`        | User profile and fasting history feature                                 |
| `:features:settings`       | App settings feature                                                     |
| `:core`                    | Core models, repositories, sync services, Room DB, DataStore, constants  |

### Key patterns

- Return `Flow` from all repository methods
- Manage UI state with `StateFlow`/`SharedFlow` in ViewModels
- Use `koinViewModel()` in Compose — register in the appropriate Koin module: `AppModule`, `WearAppModule`, `SharedModule`, `DashboardModule`, `WearDashboardModule`, `ProfileModule`, `SettingsModule`
- Place shared models in `core/src/main/java/.../core/models`; repository interfaces in `core/src/main/java/.../core/data/repositories`

### Data sync

- Path: `/fasting_state` — keys: `is_fasting`, `start_time`, `fasting_goal`, `update_timestamp`
- Path: `/smart_reminder` — keys: `suggested_time`, `reasoning`, `timestamp`
- `BaseFastingListenerService` (shared) is extended by `FastingStateListenerService` (app) and `WatchFastingStateListenerService` (wear)

### Domain facts

- Fasting goals: `circadian` (13h), `16:8` (16h), `18:6` (18h), `20:4` (20h), `36hour` (36h)
- Minimum completed fast: 13 hours
- Spanish localization (`values-es`) in all three source modules

## Commands

```bash
./gradlew app:assembleDebug              # Build phone app
./gradlew wear:assembleDebug             # Build wear app
./gradlew app:installDebug               # Install phone app on device
./gradlew wear:installDebug              # Install wear app on device
./gradlew clean                          # Clean build
```

## Pull Requests

- Use conventional commits format (`feat`, `fix`, `chore`, etc.) with optional scope, e.g. `feat(sync): add conflict resolution`
- Keep the title under 70 characters; use the description for details
- Brief description of what changed and why — no test plan section
- Do not add "Co-Authored-By" lines
- End the PR body with hashtags: `#vibe-coded`, the AI tool used (e.g. `#claude-code`, `#gemini-cli`, `#codex`), and a tag for the feature (e.g. `#fasting`, `#sync`, `#widgets`)

### Changelogs

- `CHANGELOG.md` — Android phone/tablet app; `CHANGELOG_WEAROS.md` — Wear OS app
- Add customer-facing changes to the `[Unreleased]` section using categories: Added, Changed, Fixed, Removed
- **Include**: features, bug fixes, UX changes, performance improvements
- **Exclude**: refactoring, dependency bumps, architecture changes, code style

## Reference Docs

@docs/ARCHITECTURE.md
@docs/DATA_SYNC.md

---
> Source: [charliesbot/one](https://github.com/charliesbot/one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
