## pt-budget

> This is a 100% client-side personal finance and project-cost tracking application built with Next.js, React, TypeScript, Material UI, and SQLite WebAssembly. There is no application backend. Database execution, persistence, and cross-tab coordination happen in the browser through a SharedWorker.

# Budget Tracker - AI Coding Instructions

## Project Overview

This is a 100% client-side personal finance and project-cost tracking application built with Next.js, React, TypeScript, Material UI, and SQLite WebAssembly. There is no application backend. Database execution, persistence, and cross-tab coordination happen in the browser through a SharedWorker.

## Current App Composition

### Runtime Shell

- The app uses the Next.js App Router with a client-only shell.
- [`src/app/layout.tsx`](../src/app/layout.tsx) wraps the app with `LoggingProvider`, `CustomThemeProvider`, and `DatabaseProvider`.
- [`src/app/page.tsx`](../src/app/page.tsx) dynamically loads the main shell to avoid SSR issues with browser-only database features.
- [`src/components/common/Layout/AppContent.tsx`](../src/components/common/Layout/AppContent.tsx) owns top-level navigation between dashboard, projects, trips, accounts, transactions, SQL Query, settings, and developer console.

### UI and Design Notes

- The visual system is currently driven by [`src/theme/theme.tsx`](../src/theme/theme.tsx), which defines the Material UI theme, palette, typography, elevation, and common component overrides.
- Preserve the existing Material UI visual language unless the task explicitly asks for a redesign.
- The app is responsive through Material UI breakpoints, with drawer-based mobile navigation and app-bar navigation on larger screens.
- Logging is always captured through [`src/contexts/LoggingContext.tsx`](../src/contexts/LoggingContext.tsx), and the Developer Console is a real part of the product surface rather than a temporary debug tool.

## Current Architecture

### Database and State Layers

The current database flow is:

```text
React components
    -> feature-facing slice hooks in src/contexts/useDatabaseSlices.ts
    -> domain hooks and provider contexts in src/contexts/DatabaseContext.tsx
    -> internal provider hooks in src/contexts/useDatabase*.ts
    -> src/lib/databaseService.ts
    -> src/lib/databaseWorkerService.ts
    -> public/database-worker.js
```

### Layer Responsibilities

1. **Types layer**: [`src/types/database.ts`](../src/types/database.ts) contains the shared entity contracts.
2. **Business layer**: [`src/lib/databaseService.ts`](../src/lib/databaseService.ts) and [`src/lib/sqlQueries.ts`](../src/lib/sqlQueries.ts) contain SQL orchestration, mapping, validation, analytics helpers, duplicate detection, and import/export related logic.
3. **Transport layer**: [`src/lib/databaseWorkerService.ts`](../src/lib/databaseWorkerService.ts) handles worker messaging, status, timeouts, and reconnect behavior.
4. **Execution layer**: [`public/database-worker.js`](../public/database-worker.js) runs the SQLite worker and persistence integration.
5. **Provider state layer**: [`src/contexts/DatabaseContext.tsx`](../src/contexts/DatabaseContext.tsx) composes initialization, lifecycle, cached collections, diagnostics, and domain mutations.
6. **Provider internals**:
    - [`src/contexts/useDatabaseInitialization.ts`](../src/contexts/useDatabaseInitialization.ts)
    - [`src/contexts/useDatabaseCollectionsState.ts`](../src/contexts/useDatabaseCollectionsState.ts)
    - [`src/contexts/useDatabaseTransactionSlice.ts`](../src/contexts/useDatabaseTransactionSlice.ts)
    - [`src/contexts/useDatabaseAccountManagementSlices.ts`](../src/contexts/useDatabaseAccountManagementSlices.ts)
    - [`src/contexts/useDatabaseCatalogAndPlanningSlices.ts`](../src/contexts/useDatabaseCatalogAndPlanningSlices.ts)
7. **Component adapter layer**: [`src/contexts/useDatabaseSlices.ts`](../src/contexts/useDatabaseSlices.ts) exposes narrow hooks for the app shell and feature components.

### Preferred Consumption Pattern

- Prefer the thin hooks in [`src/contexts/useDatabaseSlices.ts`](../src/contexts/useDatabaseSlices.ts) when working in components.
- Prefer the dedicated domain hooks exported from [`src/contexts/DatabaseContext.tsx`](../src/contexts/DatabaseContext.tsx) over `useDatabaseContext()` when you need provider data directly.
- Treat `useDatabaseContext()` as a compatibility aggregator, not the default path for new component code.
- Do not add startup refresh effects in components unless there is a local cache outside provider state; the provider already owns initialization and shared refresh behavior.

## Development Patterns

### WA-SQLITE Core API Functions
- Capabilities for WA-SQLITE are documented at: https://rhashimoto.github.io/wa-sqlite/docs/index.html, please refer to this for any questions about the underlying database API and capabilities.

### Adding or Changing Database Operations

When adding a new database-backed operation:

1. Update interfaces in [`src/types/database.ts`](../src/types/database.ts) if the domain contract changes.
2. Add or adjust SQL in [`src/lib/sqlQueries.ts`](../src/lib/sqlQueries.ts).
3. Implement the business operation in [`src/lib/databaseService.ts`](../src/lib/databaseService.ts).
4. Update [`src/lib/databaseWorkerService.ts`](../src/lib/databaseWorkerService.ts) if a new worker message or transport behavior is required.
5. Update [`public/database-worker.js`](../public/database-worker.js) if the worker must understand a new operation.
6. If cached collections or refresh behavior change, update [`src/contexts/useDatabaseCollectionsState.ts`](../src/contexts/useDatabaseCollectionsState.ts).
7. Put new mutation logic in the appropriate provider-domain hook instead of expanding `DatabaseContext.tsx` again.
8. Expose the new behavior through the correct slice hook in [`src/contexts/useDatabaseSlices.ts`](../src/contexts/useDatabaseSlices.ts) when components need it.
9. Add or update focused tests for the touched slice, service, or UI flow.

### Context and Provider Conventions

- `DatabaseProvider` owns initialization and renders the setup gate through `DatabaseInitializationGate`.
- Cached collections are provider-backed state, not ad hoc component state.
- Transaction and account-management mutations already refresh their provider-backed collections on success; avoid redundant manual refresh calls unless you are synchronizing a separate local cache.
- When changing provider shape or slice mapping, add or update focused tests in the context test files under `src/contexts/`.

### Feature and Component Conventions

- Components are organized by feature under `src/components/`.
- Keep new components in the existing feature folders such as `accounts`, `csv-import`, `dashboard`, `data-management`, `projects`, `settings`, `setup`, `sql-query`, `transactions`, and `trips`.
- The SQL Query page is intentionally read-only and should only execute `SELECT` statements.
- Real database mutations in UI tests or manual validation should happen through supported product flows such as transactions, trips, projects, accounts, or data management dialogs.
- Preserve the current responsive app shell patterns in `AppContent` unless the task is explicitly about navigation or layout redesign.

## Product-Specific Behavior

### Transaction and Planning Rules

- Amounts use the existing sign convention: negative for expenses and positive for income.
- Transactions can be associated with categories, companies, projects, and trips.
- Duplicate detection is hash-based and built from account, date, amount, and description data.
- Projects and trips are first-class planning surfaces, not secondary metadata.

### Import, Sample Data, and Diagnostics

- CSV import uses [`src/lib/csvImportService.ts`](../src/lib/csvImportService.ts) and the `src/components/csv-import/` UI.
- Sample data support exists through [`src/lib/sampleDataService.ts`](../src/lib/sampleDataService.ts) and `public/sample-data/`.
- The current sample data bootstrap path is triggered by `?loadSampleData` during new database creation.
- Storage/export and diagnostics are part of the normal product workflow; check dashboard, settings, and developer console surfaces before inventing new debug-only paths.

## Development Setup

### Core Commands

```bash
npm run dev
npm run build
npm test
npm run test:unit
npm run test:unit:coverage
npm run test:unit:watch
npm run test:e2e:smoke
npm run test:e2e
```

### Current Testing Strategy

- `npm test` is the default post-change verification lane in this repo. It runs unit coverage plus the Playwright smoke lane.
- `npm run test:unit` runs the Vitest unit and component suite.
- `npm run test:unit:coverage` runs Vitest with coverage enforcement.
- `npm run test:e2e:smoke` is the default browser regression lane.
- `npm run test:e2e` is reserved for broader browser validation, especially worker recovery, persistence, cross-page, or multi-step user flows.

### Testing Notes

- Always use `http://localhost:3000` for local browser work.
- Coverage enforcement currently centers on [`src/lib/databaseService.ts`](../src/lib/databaseService.ts).
- The Add Transaction dialog account field is an unnamed Material UI select, so automated tests should target it structurally rather than by accessible name.
- Browser setup should wait for either the initialization/setup screen or the loaded app shell before deciding whether to create a new database.
- Worker recovery validation uses `window.__budgetTrackerTestApi.disconnectWorker()` and a full page reload.

## Regression Testing Agent

Use the exact agent name `Budget Tracker Regression Tester` when the user asks for regression testing, validation before merge, smoke tests, coverage checks, Playwright verification, or to investigate a likely test regression.

### When to Delegate

- The user asks to validate a change, run regression coverage, or confirm nothing broke.
- The change touches worker behavior, persistence, import/export, initialization, or cross-page flows.
- The user wants a narrow failing test isolated before broader verification.

### How to Trigger It

- If the user explicitly names the agent, invoke `Budget Tracker Regression Tester` directly.
- If the user asks for regression testing without naming an agent, route to `Budget Tracker Regression Tester` automatically.
- Pass the changed file, feature, or workflow in the handoff prompt so the agent can choose the narrowest lane first.

### Example Handoffs

- `Use Budget Tracker Regression Tester to validate changes in src/lib/databaseService.ts with the narrowest reliable test lane.`
- `Run Budget Tracker Regression Tester on the trips workflow and escalate to full e2e only if smoke coverage is insufficient.`
- `Use Budget Tracker Regression Tester to verify this pull request before merge.`

## Common Gotchas

- This app is client-only. Check for browser-only APIs and SSR boundaries before adding runtime logic.
- SharedWorker support is required for the primary experience.
- Avoid broad refactors in the database provider when a change belongs in one of the internal slice hooks.
- Do not assume the SQL Query page can be used for mutations.
- Large imports and worker-heavy flows should keep expensive work off the main UI thread.
- If a test or manual validation depends on export backup fidelity, verify the current worker/export implementation first rather than assuming backup flows are healthy.

---
> Source: [TheUniquePaulSmith/pt-budget](https://github.com/TheUniquePaulSmith/pt-budget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
