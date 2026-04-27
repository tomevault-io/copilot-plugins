## oybc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OYBC (On Your Bingo Card) — An offline-first, gamified task management app that turns goals into interactive bingo boards. Multi-platform (iOS native + web) with seamless offline functionality and background sync.

**Core Architecture**: Local-first design where local databases (GRDB on iOS, Dexie on web) are the source of truth, with Firestore providing background sync for multi-device support only.

## Code Quality Standards

- Type hints and docstrings required for all functions and classes. Public APIs must document parameters, return values, and exceptions.
- Functions must be focused and small. PascalCase for classes, camelCase for functions/variables.
- Follow existing patterns within the app exactly.
- **Reuse before creating**: Search for existing utilities/components before writing new code.
- **Extract at three**: Same logic in 3+ places → extract to shared utility.
- **Share across platforms**: Constants, formatting, and validation that must be consistent across web and iOS should be centralised in a single definition.
- Proper error handling and logging for all database and network interactions.
- Evaluate third-party libraries carefully before importing — must be necessary, well-maintained, and not bloat app size.

## Testing Standards

- All new features and bug fixes must include unit tests covering core logic.
- Aim for at least 80% code coverage in all packages.
- Use Jest for TypeScript tests and XCTest for Swift tests.
- Tests should be deterministic and not rely on external services or network calls.

## Feature Implementation Guidelines

**Playground-first, one feature at a time, user-driven.** Use `/feature` skill for the full workflow.

**Core Principles**:

- ONE feature at a time — user picks what to build.
- Playground before integration — no production code without explicit user approval.
- **Build real components, not demos** — Playground features must use production-ready reusable components from `components/` / `Views/Components/`. The Playground is a testing harness, not a place for throwaway inline UI. If a component doesn't exist yet, create it as a reusable component first, then use it in the Playground.
- Mirror file structure — every playground feature is a separate file on both platforms. Container views stay thin.
- Reuse existing infrastructure — shared constants in `playgroundUtils.ts` / `PlaygroundUtils.swift`, reusable components in `components/playground/` / `Views/Components/`.

**Standard Development Process**: Ask clarifying questions first. Create a branch per feature/bugfix. TDD approach. Plan before implementing.

**Workflow Skills**:

- `/feature` — New feature development (Playground-first, Two-Gate)
- `/bugfix` — Diagnose and fix bugs (systematic debugging, plugin verification)
- `/refactor` — Restructure code without changing behavior (safety verification)
- `/integrate` — Move approved Playground features into production (Two-Gate, highest risk)

## Agent Guidelines

- Simple features (< 100 lines): single agent. Don't over-agent.
- Complex cross-platform features: use `cross-platform-coordinator` to orchestrate `react-web-implementer` (web) + `steve-jobs` (iOS).
- **Two-Gate System**: Gate 1 (plan approval by user) and Gate 2 (final review by user after automated verification). Both mandatory.
- **Plugin-driven verification** replaces manual agent reviews:
  - **Serena**: Code navigation, scope verification (compare implemented symbols vs plan), reusable component discovery.
  - **Playwright**: Mandatory web UI validation — navigate playground, interact, screenshot. Saves to `.playwright-mcp/`.
  - **Context7**: Library documentation lookups during implementation.
- Before delivery: verify BOTH platforms compile, run Playwright validation (web), use `superpowers:verification-before-completion`.
- iOS uses XcodeGen — after adding new `.swift` files, run `xcodegen generate` to regenerate the Xcode project.

### Agents

| Agent                        | Purpose                                                                |
| ---------------------------- | ---------------------------------------------------------------------- |
| `cross-platform-coordinator` | Orchestration, scope control, spec compliance, consistency enforcement |
| `react-web-implementer`      | Web (React) implementation                                             |
| `steve-jobs`                 | iOS (Swift/SwiftUI) implementation                                     |
| `sync-specialist`            | Offline-first sync (Phase 3)                                           |
| `system-design-engineer`     | Complex technical design decisions                                     |
| `ultrathink-debugger`        | Deep root cause analysis for hard bugs                                 |

## Cross-Platform File Structure

Web and iOS must mirror each other's directory and file structure.

```
Web                                                  iOS
apps/web/src/                                        apps/ios/OYBC/
├── pages/
│   └── Playground.tsx          ←→                  Views/PlaygroundView.swift
│       (container only — no feature logic)              (container only)
├── components/
│   └── playground/
│       ├── playgroundUtils.ts  ←→                  Views/Playground/PlaygroundUtils.swift
│       ├── BoardTaskSelectionPlayground.tsx ←→      Views/Playground/BoardTaskSelectionPlayground.swift
│       ├── BoardGeneratorPlayground.tsx ←→          Views/Playground/BoardGeneratorPlayground.swift
│       ├── UnifiedTaskCreatorPlayground.tsx ←→      Views/Playground/UnifiedTaskCreatorPlayground.swift
│       ├── TaskSquareActionsPlayground.tsx ←→       Views/Playground/TaskSquareActionsPlayground.swift
│       ├── CrossBoardRollupPlayground.tsx ←→        Views/Playground/CrossBoardRollupPlayground.swift
│       ├── SubtaskDerivationPlayground.tsx ←→       Views/Playground/SubtaskDerivationPlayground.swift
│       └── CompositeTaskForm.tsx ←→                 Views/Playground/CompositeTaskFormView.swift
│
└── components/                                     Views/Components/
    ├── Navbar.tsx
    ├── BingoBoard.tsx          ←→                  BingoBoard.swift
    ├── BingoSquare.tsx         ←→                  BingoSquare.swift
    ├── InteractiveTaskSquare.tsx ←→                 InteractiveTaskSquareView.swift
    ├── TypeBadge.tsx           ←→                  TypeBadgeView.swift
    ├── FilterTabs.tsx          ←→                  FilterTabsView.swift
    ├── TaskTypeSelector.tsx    ←→                  TaskTypeSelectorView.swift
    ├── SelectableTaskItem.tsx  ←→                  SelectableTaskItemView.swift
    ├── PoolItem.tsx            ←→                  PoolItemView.swift
    ├── SubtaskChip.tsx         ←→                  SubtaskChipView.swift
    ├── OperatorSelector.tsx    ←→                  OperatorSelectorView.swift
    ├── CounterStepper.tsx      ←→                  CounterStepperView.swift
    ├── ProgressStepRow.tsx     ←→                  ProgressStepRowView.swift
    ├── CountingStepFields.tsx  ←→                  CountingStepFieldsView.swift
    ├── CountingDerivationPanel.tsx ←→               CountingDerivationPanelView.swift
    ├── ProgressDerivationPanel.tsx ←→               ProgressDerivationPanelView.swift
    ├── CompositeDerivationPanel.tsx ←→              CompositeDerivationPanelView.swift
    ├── AuthGate.tsx               ←→               Views/AuthGateView.swift
    └── BoardCreatorPanel.tsx      ←→               Views/Components/BoardCreatorPanelView.swift
│
├── firebase/                                       Services/
│   ├── config.ts                  ←→               OYBCApp.swift (FirebaseApp.configure)
│   ├── authService.ts             ←→               Services/AuthService.swift
│   ├── AuthContext.tsx            ←→               (AuthService is @ObservableObject)
│   ├── syncService.ts             ←→               Services/SyncService.swift
│   └── conflictResolver.ts       ←→               (inline in SyncService.swift)
│
├── components/playground/
│   └── SyncSimulationPlayground.tsx ←→             Views/Components/SyncDashboardView.swift
```

**Rules**:

1. Container views stay thin — no form logic, no state, no database calls.
2. One file per playground feature, one file per reusable component.
3. Web: `[Name].tsx`. iOS: `[Name]View.swift` (views) or `[Name]Playground.swift` (features).
4. iOS: After adding new Swift files, run `xcodegen generate` to regenerate the Xcode project.
5. Verify both platforms have matching files before claiming completion.
6. **No single-platform commits for shared features.** Every commit that changes user-visible behaviour, shared types, hooks, or services must either (a) land the paired change on both platforms in the same commit, or (b) explicitly justify the gap in the commit message and record the platform-parity follow-up in `CLAUDE.md` (or a tracked note). "I'll do the other platform next" is exactly the drift mode to avoid — during back-to-back UI iterations it is easy to rack up web-only changes and discover hours later that iOS is several commits behind. If you catch yourself doing this, stop and mirror before continuing.

## Commands

### Monorepo (Root)

```bash
pnpm install    # Install all dependencies
pnpm build      # Build all packages
pnpm test       # Run all tests
pnpm lint       # Lint all packages
pnpm clean      # Clean all build artifacts
```

### Shared Package (`packages/shared`)

```bash
cd packages/shared
pnpm build          # Build types and validation
pnpm dev            # Watch mode
pnpm test           # Run tests
pnpm test:watch     # Watch mode for tests
pnpm test:coverage  # Coverage report
```

### Web App (`apps/web`)

```bash
cd apps/web
pnpm dev        # Dev server (http://localhost:5173)
pnpm build      # Production build
pnpm preview    # Preview production build
pnpm typecheck  # Type checking
pnpm lint       # Lint
```

### iOS App (`apps/ios`)

```bash
open OYBC.xcodeproj          # Open in Xcode
# Build: ⌘R  |  Tests: ⌘U
xcodegen generate            # Regenerate project from project.yml
xcodebuild -scheme OYBC build  # CLI build (simulator)
```

### Firebase

```bash
firebase login                           # Authenticate CLI (one-time)
firebase deploy --only firestore:rules   # Deploy security rules
```

**Setup**: Firebase config is in `.env.local` (web, gitignored) and `GoogleService-Info.plist` (iOS, gitignored). Each developer must obtain these from the Firebase console.

## Architecture

### Monorepo Structure

```
oybc/
├── apps/
│   ├── ios/           # SwiftUI + GRDB 6.24 (SQLite), iOS 17+, XcodeGen (project.yml)
│   │                  # + Firebase Auth/Firestore (SPM)
│   └── web/           # React 18 + Vite + Dexie (IndexedDB) + React Router
│                      # + Firebase JS SDK (auth, firestore, sync)
├── packages/
│   └── shared/        # TypeScript types, algorithms, validation
├── firestore.rules    # Firestore security rules (deployed via firebase CLI)
└── docs/              # Architecture documentation
```

### Offline-First Design Principle

**Critical**: Local databases are the **source of truth**, NOT Firestore.

**Data Flow**: User action → Update local DB (< 10ms) → UI updates immediately → Queue sync → Background sync to Firestore when online.

**NOT**: ~~User action → Network request → Wait → Update UI~~

### Database Schema (Identical Across Platforms)

**Tables**: `users`, `boards`, `tasks`, `task_steps`, `board_tasks`, `progress_counters`, `sync_queue`

**Key Design Elements**:

- UUID primary keys (client-generated, enables offline creation)
- Version fields (optimistic locking for conflict resolution)
- Soft deletes (`isDeleted` flag, never hard delete)
- ISO8601 timestamps
- Denormalized stats (`board.completedTasks` for instant reads)

### Type System

**`packages/shared`** is the single source of truth for types: `Board`, `Task`, `TaskStep`, `BoardTask`, `ProgressCounter`, `User`, `SyncQueueItem`. Includes Zod validation schemas and enums (`BoardStatus`, `TaskType`, `Timeframe`, `CenterSquareType`).

- **iOS**: Swift models mirror TypeScript types using GRDB's `Codable`/`FetchableRecord`/`PersistableRecord`. JSON arrays stored as strings in SQLite.
- **Web**: Dexie uses TypeScript types directly from `@oybc/shared`. Compound indexes match iOS GRDB indexes.

### Sync Strategy

**Conflict Resolution** (MVP): Last-write-wins using version fields. Higher version wins; same version → newer timestamp wins.

**Cross-Board Features**: Achievement squares and bingo lines always recomputed from source data. Task step linking uses additive merge.

See `docs/SYNC_STRATEGY.md` for details.

## Key Conventions

### iOS (Swift)

```swift
// Read
let boards = try AppDatabase.shared.fetchBoards(userId: userId)
// Write
try AppDatabase.shared.saveBoard(board)
// Transaction
try AppDatabase.shared.write { db in
    try task.save(db)
    try steps.forEach { try $0.save(db) }
}
```

- JSON arrays stored as JSON strings with custom `Codable` encode/decode
- Don't store derived values — compute from stored values

### Web (TypeScript)

```typescript
// Read
const boards = await fetchBoards(userId);
// Reactive queries
const boards = useBoards(userId); // useLiveQuery from dexie-react-hooks
// Fast compound index query
db.boards.where("[userId+isDeleted]").equals([userId, false]);
// Transaction
await db.transaction("rw", [db.tasks, db.taskSteps], async () => {
  await db.tasks.add(task);
  await Promise.all(steps.map((s) => db.taskSteps.add(s)));
});
```

### Shared Package

- No platform-specific code (no GRDB, Dexie, Firebase, React, SwiftUI)
- Only pure TypeScript: types, algorithms, validation, constants

## Important Patterns

- **Optimistic Updates**: Always update local DB first, then queue sync in background.
- **Soft Deletes**: Never hard delete. Use `isDeleted=true, deletedAt=timestamp`.
- **Denormalized Stats**: Update stats when source data changes (e.g., `board.completedTasks`).
- **Version Increment**: Always increment `version` field on updates (critical for conflict resolution).

## Common Pitfalls

- **Don't skip verification gates**: Both gates (plan approval + final review) are mandatory. Playwright validation is mandatory for web changes.
- **Don't skip the Playground**: ALL features go through Playground first with explicit user approval.
- **Don't implement multiple features at once**: ONE at a time, user-directed.
- **Don't copy from old code**: If archived/legacy code exists, it's reference only.
- **Don't use Firestore as primary storage**: Local DB is source of truth.
- **Don't hard delete**: Always soft delete for sync compatibility.
- **Don't trust denormalized values during conflicts**: Recompute from source data.
- **Don't block UI for sync**: All sync operations must be background/async.
- **Counting task field order**: Action → Max Count → Unit (not Action → Unit → Max Count).
- **Counting task title**: Optional and auto-generated from `action + maxCount + unit` if blank. Use `generateCounterTaskTitle()` from `@oybc/shared`. Not required like normal task titles.
- **Progress task step auto-creation**: When a progress task is created, each step automatically gets a standalone `Task` record linked via `TaskStep.linkedTaskId`. This makes steps immediately available as pool-addable tasks and enables cross-board rollup. Applies to `createTask()` (web), playground write blocks (iOS), and `CompositeTaskForm` inline progress subtasks.

## Performance Targets

- Local reads/writes: < 10ms
- Bingo detection: < 50ms
- Cross-board queries: < 200ms
- Sync: background only, never block UI

## Documentation

- `docs/ARCHITECTURE.md` — Technical plan, development phases
- `docs/OFFLINE_FIRST.md` — Offline-first design and data flow
- `docs/superpowers/specs/` — Feature design specs (created during `/feature` planning phase)
- `docs/SYNC_STRATEGY.md` — Conflict resolution patterns
- `docs/TASK_SYSTEM.md` — Comprehensive task system documentation
- `docs/COMPOSITE_TASKS.md` — Composite/progress task system details

**Not yet configured**: CI/CD, Prettier, SwiftLint. Linting is ESLint only (web).

## Development Status

**Phase 1**: Local Database Setup — COMPLETE
**Phase 1.5**: Working App Infrastructure — COMPLETE (web + iOS + Playground + BingoSquare)

**Phase 2**: Core Game Loop — COMPLETE (all features playground-tested)

| #   | Feature                                | Status   |
| --- | -------------------------------------- | -------- |
| 1   | 5x5 Bingo Board Grid                  | COMPLETE |
| 2   | Different Board Sizes (3x3, 4x4, 5x5) | COMPLETE |
| 3   | Bingo Detection Logic                 | COMPLETE |
| 4   | Board Randomization                   | COMPLETE |
| 5   | Center Space Logic                    | COMPLETE |
| 6   | Tasks & Task Creation                 | COMPLETE |
| 7   | Celebrations & Polish                 | SKIPPED  |

Playground-tested features: unified task creator, composite tasks, board generator, task square actions, subtask system (SF1-SF4), board task selection, cross-board rollup, board lifecycle (creation → activation → bingo detection → greenlog), timeboxed boards (calendar boundaries, expiry), uncomplete cascade.

**Phase 3**: Authentication & Sync Layer — COMPLETE

| Feature                    | Status   |
| -------------------------- | -------- |
| Firebase Auth (email/pw)   | COMPLETE |
| Google Sign-In             | COMPLETE |
| Sign in with Apple         | COMPLETE |
| Firestore sync (push/pull) | COMPLETE |
| LWW conflict resolution    | COMPLETE |
| Sync queue integration     | COMPLETE |
| Firestore security rules   | COMPLETE |
| Sync playground section    | COMPLETE |

**Current Phase**: Phase 4 — Production Integration

### Production Integration Plan

Tab-based app with auth gate. Web + iOS built simultaneously.

**Navigation**: Bottom tab bar — Boards (default), Create, Profile.

| Phase | Feature | Status |
| ----- | ------- | ------ |
| 0 | Synced user preferences (7 preference fields → Firestore) | COMPLETE |
| 1 | Auth shell + tab bar (replace "Hello OYBC" with auth-gated tabs) | COMPLETE |
| 2 | Board list (filtering, progress indicators, tap to navigate) | COMPLETE |
| 3 | Board play (bingo grid, task completion, flash messages) | COMPLETE |
| 4 | Create tab (task pool + BoardCreatorPanel) | COMPLETE |
| 5 | Profile + settings + polish (board defaults, theme, sync status, sign out) | — |

**Key principle**: Reuse playground-tested components — extract from `BoardLifecyclePlayground` into production pages. Don't rebuild.

**New files per phase**:
- Phase 1: `TabBar.tsx` ↔ `MainTabView.swift`, `BoardsPage.tsx` ↔ `BoardListView.swift`, `CreatePage.tsx` ↔ `CreateView.swift`, `ProfilePage.tsx` ↔ `ProfileView.swift`, `useSyncLoop.ts`
- Phase 2: `BoardListItem.tsx` ↔ `BoardListItemView.swift`, `BoardStatusBadge.tsx` ↔ `BoardStatusBadgeView.swift`
- Phase 3: `BoardPlayPage.tsx` ↔ `BoardPlayView.swift`

**Routes (web)**: `/boards`, `/boards/:id`, `/create`, `/profile`, `/playground` (dev tool)

## Branching Strategy

- Feature branches: `feature/feature-name`
- Bugfix branches: `bugfix/bug-description`
- Merge to `dev` only after code review and passing all tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2014sheas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
