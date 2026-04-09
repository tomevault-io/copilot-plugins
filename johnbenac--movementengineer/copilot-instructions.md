## movementengineer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Movement Engineer is a fully client-side tool for exploring, authoring, and comparing structured data about social movements. The app ingests Markdown repositories following the v2.3 data specification, compiling them into runtime datasets used by the UI.

## Commands

### Build & Test
```bash
npm install              # Install dependencies (also installs Playwright browsers)
npm test                 # Full test suite: legacy check + loader tests + unit + e2e
npm run test:unit        # Vitest unit tests only (tests/unit/**)
npm run test:e2e         # Playwright e2e tests (tests/e2e/**)
npm run test:smoke       # Quick validation: unit tests + smoke e2e
npm run build            # Production build with legacy checks
```

### Running Individual Tests
```bash
# Single unit test file
npx vitest run tests/unit/app/tabs/dashboard.test.js

# Single e2e test file
npx playwright test tests/e2e/smoke.spec.js

# E2e with specific entry point
ME_E2E_ENTRY=/index.html npx playwright test tests/e2e/smoke.spec.js
```

### Development Server
```bash
node tools/dev-server.js  # Serves on http://127.0.0.1:4173
```

## Architecture

### Core Layers

1. **Data Layer** (`src/core/markdownDatasetLoader.js`): Compiles Markdown files with YAML frontmatter into normalized data. Supports GitHub and local filesystem sources. Validates references between collections.

2. **View Models** (`src/core/viewModels.js`): Transform raw collection data into shapes optimized for UI needs. Pure functions that derive display data from the snapshot.

3. **Store** (`src/app/store.js`): Central state management with pub/sub pattern. Holds `snapshot` (the data), `currentMovementId`, navigation state, and dirty flags.

4. **Shell** (`src/app/shell.js`): Tab navigation controller. Manages tab mounting/unmounting lifecycle and URL hash routing.

5. **Tabs** (`src/app/tabs/`): Feature modules that register on `window.MovementEngineer.tabs`. Each implements:
   - `render(ctx)` - required, idempotent rendering
   - `mount(ctx)` - optional, one-time setup
   - `unmount(ctx)` - optional, cleanup on tab switch

### Context Object (ctx)

All tabs receive a shared context object containing:
- `store` / `getState()` / `setState()` / `update()` - state access
- `services` - DomainService, ViewModels, MarkdownDatasetLoader, etc.
- `ui` - status display, markdown preview
- `dom` - DOM utilities (clearElement, ensureSelectOptions, etc.)
- `actions` - navigation helpers (jumpToEntity, selectMovement, etc.)
- `tabs` - registry of all tab modules

### Data Model Collections

The app works with 10 collection types, all scoped to a `movementId`:
- `movements` - top-level containers
- `textCollections` / `texts` - hierarchical text nodes with parent references
- `entities` - beings, places, objects, ideas (with `kind` field)
- `practices` - rituals, disciplines, activities
- `events` - happenings with recurrence patterns
- `rules` - norms with graduated `RuleKind`
- `claims` - boolean assertions with sources
- `media` - linked artifacts (images, audio, video)
- `notes` - annotations on any record type

**Key design principle**: One-way references. Items point to entities via ID arrays; inverse lookups are computed at render time.

### Legacy Check System

The build runs `check-no-legacy.mjs` to ensure no banned files (app.js, modules/bootstrap.mjs) or legacy strings exist. This enforces the migration from the original architecture.

## Testing Patterns

### Unit Tests (Vitest + jsdom)
- Located in `tests/unit/`
- Mirror the source structure
- Create minimal ctx objects with mocked services
- Use `vi.fn()` for service mocks

### E2E Tests (Playwright)
- Located in `tests/e2e/`
- Use `gotoApp(page)` helper from `tests/helpers/gotoApp.js`
- Dev server auto-starts via playwright config

## File Organization

```
src/app/
  main.js          # Bootstrap, ctx creation, tab registration
  store.js         # State management
  shell.js         # Tab navigation
  actions.js       # Navigation action creators
  tabs/            # Feature modules (dashboard, canon, graph, etc.)
  ui/              # Shared UI utilities (dom.js, status.js, markdown.js)
  utils/           # Value helpers

src/core/markdownDatasetLoader.js  # Data compilation (browser + Node)
src/core/viewModels.js              # Data -> UI transformation
src/models/comparisonModel.json     # Movement comparison schema
src/runtime/storageService.js       # LocalStorage persistence
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johnbenac)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/johnbenac)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
