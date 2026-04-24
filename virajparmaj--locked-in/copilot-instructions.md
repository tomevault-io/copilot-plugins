## locked-in

> - **Name:** locked-in

# CLAUDE.md — LockedIn

## Project Overview
- **Name:** locked-in
- **Description:** macOS menu-bar Electron app that automates LinkedIn DMs using your logged-in browser session via AppleScript
- **Status:** Active
- **Owner:** Veer

---

## Tech Stack
- **Language:** TypeScript (strict)
- **Framework:** Electron 33 + React 18 + Vite (via electron-vite 4)
- **UI Library:** shadcn/ui
- **Styling:** Tailwind CSS 3.4
- **Database:** SQLite (better-sqlite3) — stored at `app.getPath('userData')/lockedin.db`
- **Scheduler:** node-schedule (in-memory cron jobs)
- **macOS Automation:** AppleScript via `osascript` (controls Chrome/browser)
- **Testing:** Vitest 4.1
- **Icons:** Lucide React
- **IDs:** nanoid
- **Dates:** date-fns
- **Package Manager:** npm

---

## Architecture

### Directory Structure
```text
locked-in/
├── electron/                   # Electron main process
│   ├── main.ts                 # App init, tray, window, power events
│   ├── preload.ts              # IPC bridge (window.api)
│   ├── ipc/                    # IPC handlers per domain
│   │   ├── contacts.ipc.ts
│   │   ├── reminders.ipc.ts
│   │   ├── settings.ipc.ts
│   │   └── ...
│   ├── services/               # Core business logic
│   │   ├── db.service.ts       # SQLite schema, queries, migrations
│   │   ├── scheduler.service.ts # Job scheduling + execution engine
│   │   ├── linkedin.service.ts # Browser automation (AppleScript)
│   │   └── reminder.service.ts # Reminder scheduling + notifications
│   └── utils/
│       ├── applescript.ts      # AppleScript runner
│       └── logger.ts           # Structured logging
├── src/                        # React renderer
│   ├── main.tsx                # React DOM entry
│   ├── App.tsx                 # Tabbed UI (6 tabs)
│   ├── pages/                  # Dashboard, Contacts, Reminders, Calendar, Logs, Settings
│   ├── components/             # Forms, modals, ErrorBoundary, ui/ (shadcn)
│   ├── contexts/               # ContactContext, ScheduleContext, ReminderContext
│   ├── hooks/                  # useSettings, useLogs
│   └── lib/
│       ├── ipc.ts              # IPC proxy with 10s timeout
│       └── utils.ts            # Date formatting, slug extraction
├── shared/                     # Shared between main + renderer
│   ├── types.ts                # All TypeScript interfaces
│   └── linkedin.ts             # LinkedIn URL validation & parsing
├── tests/                      # Vitest test files
├── docs/                       # Architecture docs
└── resources/                  # App icons, tray icons
```

### Key Entry Points
- **Electron main:** `electron/main.ts` — app lifecycle, tray, IPC registration, scheduler/reminder init
- **React entry:** `src/main.tsx` → `src/App.tsx` (6-tab layout with context providers)
- **IPC bridge:** `electron/preload.ts` exposes `window.api` (typed as `ElectronAPI`)
- **Config:** `electron-vite.config.ts`, `tsconfig.json`, `tailwind.config.ts`

### Data Flow
```
React UI → IPC proxy (src/lib/ipc.ts) → preload bridge → IPC handlers (electron/ipc/)
  → services (db/scheduler/linkedin/reminder) → SQLite DB + AppleScript automation
```
- Scheduler & reminder services emit events back to renderer (`schedule:executed`, `reminder:triggered`)
- Frontend listens via `window.api.on(...)` and refreshes context state

---

## Development Setup

### Prerequisites
- macOS (Apple Silicon) — AppleScript automation requires macOS
- Node.js
- Chrome (or configured browser) must be logged into LinkedIn

### Install & Run
```bash
npm install
npm run rebuild          # Rebuild better-sqlite3 native module for Electron
npm run dev              # Start Electron + Vite with hot reload
```

### All Commands
```bash
npm run dev              # Dev mode with hot reload
npm run build            # Build to ./out/
npm run dist             # Build + create .app + .dmg
npm run dist:dmg         # Build + DMG only
npm test                 # Run tests once
npm run test:watch       # Watch mode
npm run typecheck:web    # TS check renderer
npm run typecheck:node   # TS check main process
npm run verify           # Tests + both typechecks
```

### Configuration
No `.env` files — all settings are stored in SQLite `settings` table. Defaults:
- `globalDryRun: false`, `sendDelayMs: 5000`, `pageLoadDelayMs: 4000`
- `browserApp: 'Google Chrome'`, `maxRetries: 2`
- `minIntervalBetweenSends: 60000` (1 min between sends)

---

## Database Schema (SQLite)

| Table | Purpose |
|-------|---------|
| `contacts` | id, name, linkedin_url, linkedin_slug, company, notes, tags |
| `schedules` | id, contact_id (FK), message, schedule_type, timing fields, enabled, dry_run |
| `run_logs` | id, schedule_id (FK), status, error_message, timing, retry info |
| `settings` | key-value store for app configuration |
| `notification_rules` | id, contact_id (FK), frequency, next_reminder_at, enabled |

**Indexes:** `contacts(linkedin_slug)`, `run_logs(schedule_id)`, `run_logs(fired_at DESC)`

Schema lives in `electron/services/db.service.ts` — migrations run on app startup.

---

## Coding Standards

### Naming Conventions
- Files: `kebab-case` for utils/services, `PascalCase` for React components
- Components: `PascalCase`
- Functions/variables: `camelCase`
- Types/interfaces: `PascalCase` (no prefix)
- DB columns: `snake_case`

### File Organization
- One component per file in `src/components/`
- Pages in `src/pages/` (route = tab name)
- Shared types in `shared/types.ts`
- Utilities in `src/lib/`
- Custom hooks in `src/hooks/`
- IPC handlers in `electron/ipc/` (one file per domain)
- Services in `electron/services/`

---

## Reusable Utilities (use these, don't reinvent)
- `src/lib/ipc.ts` — typed IPC proxy with 10s timeout, wraps all `window.api` calls
- `src/lib/utils.ts` — `cn()` helper, `formatDate()`, `getScheduleTypeLabel()`, timeline bucketing
- `shared/types.ts` — all interfaces: `Contact`, `Schedule`, `Reminder`, `RunLog`, `AppSettings`, `ElectronAPI`
- `shared/linkedin.ts` — `extractLinkedInSlug()`, `normalizeLinkedInProfileUrl()`, `isLinkedInProfileUrl()`, `normalizeLinkedInRecipientText()`
- `electron/utils/logger.ts` — structured logging for main process
- `electron/utils/applescript.ts` — AppleScript execution wrapper

---

## State Management
- **Primary:** React Context API (3 contexts)
  - `ContactContext` — CRUD + search for contacts
  - `ScheduleContext` — CRUD + fire times + test send
  - `ReminderContext` — CRUD + snooze
- **Hooks:** `useSettings()` for settings, `useLogs()` for execution logs
- **Cross-context refresh:** Custom event `'lockedin:contacts-changed'` triggers schedule/reminder refresh
- **Avoid:** Redux, Zustand, or any additional state library

---

## Error Handling
- **Toasts:** `useToast()` hook → variants: `default`, `destructive`, `success` (auto-dismiss 3s)
- **ErrorBoundary:** `src/components/ErrorBoundary.tsx` catches React render errors
- **IPC:** 10s timeout per call, errors logged before throwing
- **LinkedIn automation:** Non-retryable (a11y denied, rate limit, login wall) vs retryable (network, timeout). Backoff tiers: [15s, 45s, 120s]. Max retries configurable via settings.

---

## Testing
- **Framework:** Vitest 4.1 with globals enabled
- **Config:** `vitest.config.ts` — aliases `@` → `src/`, `@shared` → `shared/`
- **Test files:** `tests/*.test.ts`
- **Run:** `npm test` (once) or `npm run test:watch`
- **Existing tests:** IPC validation, LinkedIn selectors, reminder logic, scheduler logic, toast contracts

---

## Git Workflow

### Commit Format
```
<type> : <short summary 4-5 words max>

- change point 1
- change point 2
- change point 3
```
Types: `chore` | `feat` | `fix` | `docs` | `refactor` | `test` | `style`

---

## Known Issues & Tech Debt
- LinkedIn DOM selectors may break if LinkedIn updates their UI — test with `tests/linkedin-selectors.test.ts`
- AppleScript automation only works on macOS with Accessibility permissions granted
- Scheduler jobs are in-memory — app must be running to fire schedules (sleep/wake resync exists)

---

## Debugging Tips
- If LinkedIn sends fail → check Accessibility permissions in System Settings > Privacy
- If schedules don't fire → check `run_logs` table via Logs tab, verify app wasn't sleeping
- If IPC calls timeout → check main process console for errors (10s timeout in `src/lib/ipc.ts`)
- If native module errors → run `npm run rebuild` to rebuild better-sqlite3
- Run `npm run verify` before commits to catch type errors and test failures

---

## Learned Patterns
<!-- Add patterns here when mistakes are corrected -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/virajparmaj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
