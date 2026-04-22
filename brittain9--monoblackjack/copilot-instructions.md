## monoblackjack

> Casino-grade blackjack simulator with SQLite-backed analytics. Not just a game — a data tool for strategy analysis with configurable rules, detailed stats tracking, and strategy matrix visualization.

# MonoBlackjack - Project Context

## What This Is
Casino-grade blackjack simulator with SQLite-backed analytics. Not just a game — a data tool for strategy analysis with configurable rules, detailed stats tracking, and strategy matrix visualization.

## Architecture

**Multi-Project Clean Architecture**
```
MonoBlackjack.Core   → Domain logic, events, game rules (no dependencies)
MonoBlackjack.Data   → SQLite repositories, schema, persistence
MonoBlackjack.App    → MonoGame UI, EventBus, state machine, rendering
```

- **Event-driven:** GameRound emits events → EventBus → StatsRecorder/UI subscribe
- **Composition root:** BlackjackGame.cs wires everything up
- **State pattern:** MenuState, GameState, SettingsState, StatsState

## Tech Stack
- .NET 10, MonoGame 3.8.4.1 (DesktopGL)
- SQLite (Microsoft.Data.Sqlite)
- xUnit + FluentAssertions (94 tests, all passing)

## Build & Run
```bash
dotnet build MonoBlackjack.slnx
dotnet test MonoBlackjack.slnx
dotnet run --project src/MonoBlackjack.App
```

## Key Conventions

- **Domain purity:** Core project has zero external dependencies
- **Testability:** All game logic unit tested, rules flow via immutable `GameRules` instances
- **Events:** All state changes emit domain events (see RoundEvents.cs)
- **Config philosophy:** Configurable = real casino variations. Hardcoded = universal blackjack constants (BustNumber=21, etc.)
- **Cross-platform:** No OS-specific code, Liberation Sans font

## Coordinate System Standard
- ALL UI/game render elements use CENTER-ANCHOR positioning
- `Position` is always the element center point
- `DestRect` converts center to top-left: `(Position.X - Size.X/2, Position.Y - Size.Y/2)`
- Direct `SpriteBatch.Draw`/`DrawString` calls should explicitly convert center coordinates to top-left (or use centered origins)

## Database
- **Location:** `~/.local/share/MonoBlackjack/monoblackjack.db`
- **Schema:** Profile → Session → Round → HandResult/CardSeen/Decision
- **Rule fingerprint:** BlackjackPayout, DealerHitsS17, DeckCount, SurrenderRule stored per round
- **Migrations:** Handled in DatabaseManager.RunMigrations()

## Current Development Priority
Game rendering and feel is the **active priority**. Stats feature (`StatsState`, analytics dashboards) is stable but deprioritized — do not invest effort there until game rendering is solid. Focus: GameState rendering, card layout, table presentation, and game flow polish.

## MonoBlackjack.App Folder Structure
```
Animation/          ← Tween system (stable, don't touch)
Infrastructure/     ← Events, Input, DevTools, Settings, Diagnostics
Layout/             ← GameLayoutCalculator — pure math, testable (stable)
Rendering/          ← Sprite base classes + game/stats renderers
  Controls/         ← Button component
  Game/             ← GameTableRenderer, GameHudPresenter
  Stats/            ← All stats panel renderers (deprioritized)
Services/           ← StatsRecorder (background event handler)
States/             ← State machine screens
  Game/             ← GameAnimationCoordinator + input/pause/phase helpers
```

## Rendering System Design
**Two mechanisms in parallel (intentional):**
- `SceneRenderer` / `SpriteLayer` / `Sprite` subclasses: animated card sprites driven by TweenManager
- Direct `SpriteBatch.DrawString` / `Button.Draw`: HUD text, buttons, table

Cards need tween-driven animation; buttons/text do not. Keep them separate.

**GameState.Draw() structure:** Opens/closes SpriteBatch multiple times across phase branches (Betting, Bankrupt, Playing/Insurance/Paused). Each branch manages its own Begin/End pairs. This is intentional (different sampler states for table vs UI), but the branching logic is complex.

## Table Layout Geometry (Critical Context)
Source image: `3840×1388 px` (Art/BlackjackTable.png). Arc coordinates in `GameLayoutCalculator` were extracted directly from this image (source-space pixel coordinates).

**At 1366×768 viewport:** `scale = min(1366/3840, 768/1388) = 0.356` → table fills full viewport width with zero horizontal margin.

**Arc Y positions in screen space at 1366×768:**
- Top arc inner surface: sides at y≈230, center bottom at y≈412
- Insurance arc: sides at y≈408, center bottom at y≈487
- Dealer cards Y (arc-derived): card top at y≈255, bottom at y≈409 → cards appear *inside* the arc, not above it
- Player cards Y (arc-derived): card top at y≈491

**Known rendering bugs (as of Feb 22, 2026):**
1. Table fills viewport width — no margin above for dealer cards, below for buttons
2. Dealer cards positioned inside the arc instead of above the hot dog shape
3. "BLACKJACK PAYS" / "DEALER MUST DRAW" arc text overlaps betting controls (both at y≈50% viewport)
4. "INSURANCE PAYS 2 TO 1" is a permanent table decal but only shown during Insurance phase — wrong
5. Missing dealer peek animation when dealer's up-card is an Ace

## Important Files
- `GameRound.cs` (572 lines) — Round orchestration, all game actions (Hit/Stand/Split/Double/Surrender)
- `GameState.cs` (~896 lines) — Main gameplay UI, card rendering, button layout
- `GameAnimationCoordinator.cs` — Card sprite lifecycle, animation events, layout delegation
- `GameTableRenderer.cs` — Table texture + arc text rendering
- `GameLayoutCalculator.cs` — All layout math (card positions, button sizing, table scale)
- `GameConfig.cs` — Constants + setting keys (no mutable gameplay state)
- `StatsState.cs` — Dashboard with Overview/Analysis tabs (deprioritized)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brittain9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
