## gm-sim

> NFL GM Simulator — React Native/Expo/TypeScript. Mobile-first GM experience with hidden mechanics (Strat-O-Matic philosophy). Players never see ratings, probability tables, or sim internals. Three-layer rating system: true→perceived→effective. AsyncStorage persistence (3 save slots), 32-team league, infinite career mode. Currently ~60% complete.

# AGENTS.md

## Identity

NFL GM Simulator — React Native/Expo/TypeScript. Mobile-first GM experience with hidden mechanics (Strat-O-Matic philosophy). Players never see ratings, probability tables, or sim internals. Three-layer rating system: true→perceived→effective. AsyncStorage persistence (3 save slots), 32-team league, infinite career mode. Currently ~60% complete.

## Architecture

```
src/
├── core/           # Game engine + simulation (THE HEART)
│   ├── models/     # GameState, Player, Team, League, Coach, Scout, Owner
│   ├── engine/     # PlayResolver, MatchupResolver, EffectiveRatingCalculator
│   ├── gameflow/   # GameFlowManager, WeekProgressionService
│   ├── season/     # SeasonManager, ScheduleGenerator, StandingsCalculator
│   ├── draft/      # DraftClassGenerator, Prospect, AIDraftStrategy
│   ├── freeAgency/ # FreeAgencyManager, MarketValueCalculator
│   ├── offseason/  # OffseasonOrchestrator, 12 phase handlers
│   ├── coaching/   # CoachGenerator, SchemeDefinitions, ChemistryCalculator
│   ├── contracts/  # SalaryCapManager, CutCalculator, ExtensionSystem
│   ├── scouting/   # ScoutingDepartmentManager, BigBoardGenerator
│   ├── career/     # PatienceMeterManager, FiringMechanics, JobMarketManager
│   ├── news/       # NewsFeedManager, StoryTemplates
│   └── generators/ # PlayerGenerator, TraitGenerator, ItFactorGenerator
├── screens/        # 47 container screens (manual nav in App.tsx)
├── components/     # Reusable UI: PlayerCard, CoachCard, Gamecast
├── navigation/     # AppNavigator, GameContext, ScreenWrappers
├── services/       # NewGameService, GameStorage, WeekFlowManager
├── state/          # Zustand store (meta-state: save slots, timestamps only)
├── styles/         # Design tokens: colors, spacing, typography
└── types/          # TypeScript definitions
```

Patterns: Immutable data flow (funcs return new objects). Sim engine is black box — internal calcs separated from UI. PlayerViewModel enforces privacy boundaries.

## Core Systems Reference

### Player Model
`core/models/player/Player.ts`, `TechnicalSkills.ts`, `HiddenTraits.ts`, `ItFactor.ts`
- Physical attrs + skill RANGES narrow via scouting
- Hidden traits emerge through gameplay (clutch, injury prone, leader)
- "It" factor (1-100) NEVER exposed
- trueValue (engine) → perceivedRange (UI) → effectiveRating (game)

### Simulation Engine
`core/engine/PlayResolver.ts`, `MatchupResolver.ts`, `EffectiveRatingCalculator.ts`
- Probability-based play-by-play
- Weighted targets across depth chart, RB rotation with fatigue
- Multi-factor: pressure, coverage, distance, weather
- Scheme affects play selection

### Scouting
`core/scouting/ScoutingDepartmentManager.ts`, `ScoutAccuracySystem.ts`
- Auto regional coverage, focus player designations
- Scout accuracy HIDDEN — learned by comparing predictions
- Ranges narrow with investment

### Coaching
`core/coaching/CoachGenerator.ts`, `SchemeDefinitions.ts`, `ChemistryCalculator.ts`
- HC→OC/DC/ST hierarchy
- Schemes: West Coast, Air Raid, Power Run
- Chemistry, coaching tree lineage

### Draft
`core/draft/DraftClassGenerator.ts`, `Prospect.ts`, `CombineSimulator.ts`
- Combine measurables, pro days, private workouts
- Big board vs needs, pick trading

### Free Agency
`core/freeAgency/FreeAgencyManager.ts`, `MarketValueCalculator.ts`, `RFATenderSystem.ts`
- Three-wave structure: legal tampering, day-1 frenzy, trickle phase
- RFA tenders, compensatory pick calculation
- Market value estimation hidden from user

### Contract System
`core/contracts/SalaryCapManager.ts`, `CutCalculator.ts`, `FranchiseTagSystem.ts`
- Salary cap management (~$255M)
- Franchise/transition tags, dead money on cuts
- Guarantees, incentives, escalators

### Season Loop
`core/season/SeasonManager.ts`, `WeekSimulator.ts`, `PlayoffGenerator.ts`
- Week: schedule → sim → postgame → summary → advance
- Bye weeks handled (no blocking), standings with tiebreakers
- 14-team playoffs with seeding

### Career Mode
`core/career/PatienceMeterManager.ts`, `OwnerExpectationsSystem.ts`, `JobMarketManager.ts`
- Owner personality system (5 distinct types)
- Patience meter based on performance vs expectations
- Job market if fired, multi-team career progression

### Gamecast
`components/gamecast/Scoreboard.tsx`, `PlayByPlayFeed.tsx`, `FieldVisualization.tsx`
- Play-by-play presentation layer
- Users see results only — NEVER the math behind them
- Post-game box scores and grades (perceived, not true)

## Screen Map

**Entry (5):** Start → TeamSelection → StaffDecision → StaffHiring → GMDashboard

**Week (6):** Schedule, GameDay, PostGameSummary, WeeklyDigest, WeeklySchedulePopup, Standings

**Roster (5):** Roster, DepthChart, DepthChartV2, PlayerProfile, SinglePlayerCard

**Draft (5):** DraftBoard, DraftRoom, CombineProDay, BigBoard, CompPickTracker

**Free Agency (4):** FreeAgency, RFA, RumorMill, Trade

**Staff (5):** Staff, CoachProfile, CoachHiring, CoachingTree, StaffHiring

**Career (5):** OwnerRelations, CareerLegacy, CareerSummary, JobMarket, Interview

**Offseason (6):** Offseason, SeasonRecap, OTAs, TrainingCamp, Preseason, FinalCuts

**Other (7):** Finances, ContractManagement, News, Settings, PlayoffBracket, Stats, ScoutingReports

Manual nav via `currentScreen` state in App.tsx. All bidirectional. No orphaned screens.

## Dev Phases & PR Map

**Original PR Plan (6 phases, 25 PRs):**
- Phase 1 Foundation: PR-02 player, PR-03 staff, PR-04 team/league/owner
- Phase 2 Sim Core: PR-05 player gen, PR-06 sim engine, PR-07 game, PR-08 season
- Phase 3 Staff: PR-09 coaching, PR-10 chemistry, PR-11 scouting, PR-12 reports
- Phase 4 Player Life: PR-13 prospects, PR-14 draft, PR-15 contracts, PR-16 FA
- Phase 5 Career: PR-17 owner, PR-18 patience, PR-19 jobs, PR-20 offseason
- Phase 6 Polish: PR-21 traits, PR-22 news, PR-23 gamecast, PR-24 profiles, PR-25 tests

**Current Work (remaining ~40%):**
- Phase A: Persistence (game results, prospect flags, settings not saving)
- Phase B: Wire DraftRoom (screen exists, needs nav connection)
- Phase C: Free Agency screen wiring (backend exists)
- Phase D: Roster ops (cut/trade/extend functionality)
- Phase E: League sim (all 32 teams, injuries, stats)
- Phase F: News integration (replace hardcoded data)
- Phase G: Career & firing mechanics
- Phase H: Offseason 12-phase flow
- Phase I: Polish and cleanup

## Brand Rules (NEVER VIOLATE)

1. No `trueValue`/`trueRating`/`baseRating` exposed to any UI component
2. No single-number "overall rating" — skills shown as RANGES only
3. Hidden traits emerge through events, NEVER listed explicitly upfront
4. "It" factor is NEVER exposed or hinted at in UI
5. Sim internals (probability tables, weights, formulas) NEVER shown
6. Gamecast shows results only — play outcomes, not calculations
7. Scout accuracy is HIDDEN — user infers by comparing predictions
8. Coach tendencies affect sim but NEVER displayed as percentages
9. Use PlayerViewModel for all UI rendering to enforce privacy

## Known Issues & Active Work

**Resolved:**
- Bye week blocking (PR #90)
- Continue button on post-game (PRs #87, #90)
- Four critical bugs from audit (PR #89)
- Event-driven game flow (PR #86)

**Open (by priority):**
- CRITICAL: Game results not persisted after app close (App.tsx:592-614)
- CRITICAL: DraftRoomScreen exists but disconnected from navigation
- CRITICAL: FreeAgency shows "Coming Soon" alert
- HIGH: News uses hardcoded placeholder data
- HIGH: No trading UI (backend exists)
- HIGH: Playoff seeding never calculated
- HIGH: Injuries never triggered during games
- MEDIUM: Settings/prospect flags not persisted
- MEDIUM: Offseason 12-phase not wired

**Regression:** Run full 32-team season sim before any PR merge.

## Conventions

- TypeScript strict mode (`noImplicitAny`, `strictNullChecks`)
- React Native + Expo SDK
- Manual screen nav via `currentScreen` state in App.tsx
- AsyncStorage persistence (3 slots via GameStorage)
- Zustand for meta-state only; GameState flows via props as immutable
- Jest with setup in `src/tests/setup.ts`
- Path aliases: `@core/*`, `@state/*`, `@services/*`, `@ui/*`, `@types/*`
- Commands: `npm start`, `npm test`, `npm run lint`, `npm run typecheck`
- Boolean gate pattern for feature validation before proceeding

## Cursor Cloud specific instructions

### Services overview

This is a fully client-side React Native/Expo app with **no backend services, databases, or external APIs**. The only service to run is the Expo/Metro dev server.

### Running the app

- `yarn web` starts the Expo web dev server on port 8081 (equivalent to `yarn start --web`).
- The web target is the easiest to test in the cloud VM (no iOS/Android emulators available).
- Game generation takes ~2-3 seconds; the "New Game" flow goes: Start → TeamSelection → StaffDecision → Dashboard.

### Quality checks

All standard dev commands are documented in `CLAUDE.md`. Key ones:

| Check | Command |
|---|---|
| Lint | `yarn lint` |
| Typecheck | `yarn typecheck` |
| Tests | `yarn test` |
| Format check | `yarn format:check` |

### Node version

The project requires Node.js 20.x (>= 20.19.4 due to `@react-native/dev-middleware` engine constraint). The `.tool-versions` file says 20.19.0 but that is slightly outdated — use `nvm use 20` to pick up the latest 20.x LTS.

### Gotchas

- Yarn Classic (v1) is used — do not use `yarn set version` or Yarn Berry features.
- The full test suite (143 suites, ~4000 tests) takes ~5 minutes. Use `yarn test -- path/to/file.test.ts` for targeted runs.
- ESLint v9 with flat config is used (`eslint.config.mjs`). A few unused-disable-directive warnings are expected and non-blocking.
- Peer dependency warnings during `yarn install` (especially around Babel/core) are expected and harmless.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/j-r-j)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/j-r-j)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
