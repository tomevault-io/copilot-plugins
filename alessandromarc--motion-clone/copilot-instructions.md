## motion-clone

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Nexto** is an intelligent task scheduling app (originally a Motion clone). It is a monorepo with:

- `frontend/` — Next.js 16 App Router (port 3000)
- `backend/` — Express.js 5 REST API (port 3003)
- `shared/` — Shared TypeScript types (`shared/types.ts`)
- `e2e/` — Playwright end-to-end tests

Database: Supabase (PostgreSQL with Row Level Security). Deployed to Vercel.

## Commands

All commands are run from the **repo root** unless noted otherwise.

### Development

```bash
npm run dev          # Start frontend + backend concurrently
npm run dev-f        # Frontend only (Next.js on port 3000)
npm run dev-b        # Backend only (Express on port 3003)
```

### Building

```bash
npm run build        # Build frontend then backend
npm run build-f      # Frontend only
npm run build-b      # Backend only (tsc, run from backend/)
```

Backend build note: TypeScript compiles from the **repo root**. Output goes to `dist/backend/src/` and `dist/shared/`. Do not use path aliases for shared types — use relative imports like `../../../shared/types.js`.

### Testing

```bash
npm run test              # Run all unit tests (frontend + backend)
npm run test:frontend     # Frontend Jest tests only
npm run test:backend      # Backend Jest tests only
npm run test:e2e          # Playwright E2E tests
npm run test:coverage     # Unit tests with coverage reports
```

Run a **single backend test file**:

```bash
cd backend && NODE_OPTIONS=--experimental-vm-modules jest src/services/__tests__/taskService.test.ts
```

Run a **single frontend test file**:

```bash
cd frontend && jest src/components/Tasks/__tests__/TaskCard.test.tsx
```

Backend tests require `NODE_OPTIONS=--experimental-vm-modules` because they use ESM with `jest.unstable_mockModule`.

### Linting & Formatting

```bash
npm run lint         # ESLint on frontend/src and backend/src
npm run lint:fix     # Auto-fix lint issues
npm run format       # Prettier (write)
npm run format:check # Prettier (check)
npm run unused       # Find unused TypeScript exports
```

## Architecture

### Backend (`backend/src/`)

Express.js REST API following a layered architecture:

- **`middleware/auth.ts`** — JWT verified locally (not via remote Supabase call). Attaches `req.userId` and `req.supabaseClient` (an authenticated `SupabaseClient`) to every request. Milestones routes currently lack auth middleware (known gap).
- **`config/supabase.ts`** — `getAuthenticatedSupabase(token)` creates a client; `verifyAuthToken(token)` does local JWT verification.
- **`services/`** — Business logic. Services receive a `SupabaseClient` as a parameter rather than creating their own. Never call `getAuthenticatedSupabase()` inside a service.
- **`routes/`** — Route handlers pass `req.supabaseClient` to service methods.
- **`utils/responseHelpers.ts`** — `ResponseHelper.success()`, `.error()`, `.notFound()`, `.unauthorized()` for consistent API responses.

Service pattern:

```typescript
// Service method receives the authenticated client
async getAllTasks(client: SupabaseClient): Promise<Task[]> { ... }

// Route handler passes it
router.get('/', async (req: AuthRequest, res: Response) => {
  const tasks = await taskService.getAllTasks(req.supabaseClient);
});
```

**Auto-scheduling**: `services/autoScheduleService.ts` and `services/syncScheduler.ts` handle background task scheduling. The scheduler does not run on Vercel (checked via `process.env.VERCEL`).

### Frontend (`frontend/src/`)

Next.js App Router with the following structure:

- **`app/`** — Pages: `/`, `/tasks`, `/projects`, `/calendar`, `/profile`, `/changelog`
- **`components/`** — Organised by domain: `Tasks/`, `Calendar/`, `Projects/`, `Auth/`, `Onboarding/`, `Sidebar/`, `shared/`, `ui/` (shadcn components)
- **`contexts/AuthContext.tsx`** — Global auth state via Supabase; provides `user`, `session`, `activeSchedule`, `signInWithGoogle`, `signOut`. Supports `NEXT_PUBLIC_AUTH_BYPASS=1` for local dev without real auth.
- **`services/`** — API clients that call the backend. All use `apiClient.ts` which automatically attaches the Supabase JWT as `Authorization: Bearer <token>`.
- **`hooks/useAsyncData.ts`** — Generic hook for data fetching with loading/error state.
- **`lib/`** — `supabase.ts` (browser client), `auth.ts` (`getAuthToken()`), `analytics.ts` (PostHog), `logger.ts`.

UI stack: Tailwind CSS v4, Radix UI primitives, shadcn/ui components, `lucide-react` icons, `sonner` toasts, `react-hook-form` + `zod` for forms, `date-fns` for dates.

### Shared (`shared/types.ts`)

Single file exported by both sides. Core types: `Task`, `Project`, `Milestone`, `CalendarEvent`, `Schedule`, `WorkItemStatus`. Import via relative path only — no `@shared` alias.

### E2E Tests (`e2e/`)

Playwright tests run against the Next.js dev server with `NEXT_PUBLIC_AUTH_BYPASS=1`. API calls to `/api/*` are intercepted per-test via `page.route()` — no live backend is needed. Test files: `auth.spec.ts`, `tasks.spec.ts`, `projects.spec.ts`, `calendar.spec.ts`, `recurring-tasks.spec.ts`, `auto-schedule.spec.ts`.

## Testing Patterns

### Backend (ESM with `jest.unstable_mockModule`)

Service tests mock Supabase and dynamically import the service:

```typescript
jest.unstable_mockModule('../../config/supabase.js', () => ({
  getAuthenticatedSupabase: jest.fn().mockReturnValue(mockClient),
}));
const { TaskService } = await import('../taskService.js');
```

Route tests use `supertest` with a minimal Express app and mock `verifyAuthToken`:

```typescript
jest.unstable_mockModule('../../config/supabase.js', () => ({
  verifyAuthToken: jest
    .fn()
    .mockReturnValue({ userId: 'user-1', exp: 9999999999 }),
  getAuthenticatedSupabase: jest.fn().mockReturnValue({}),
}));
```

Reusable mock helpers live in `backend/src/__tests__/helpers/supabaseMock.ts`.

### Frontend

Tests use `@testing-library/react` with `jest-environment-jsdom`. See existing test files in `__tests__/` directories within each module.

## Environment Variables

**Backend** (`backend/.env.development.local`):

```
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_JWT_SECRET=        # Required for local JWT verification
SUPABASE_SERVICE_ROLE_KEY=
GOOGLE_CLIENT_ID=           # Optional: Google Calendar OAuth
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=http://localhost:3003/api/google-calendar/callback
FRONTEND_URL=http://localhost:3000
STRICT_AUTH_MODE=false      # Set true to also do remote Supabase verification
```

**Frontend** (`frontend/.env.local`):

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
NEXT_PUBLIC_API_URL=        # Optional; defaults to http://localhost:3003/api in dev
NEXT_PUBLIC_AUTH_BYPASS=1   # Set to bypass Supabase auth (local dev / E2E tests)
```

# Backend Scheduling Architecture & Process

This document provides a detailed overview of the auto-scheduling engine in the backend, explaining how tasks are transformed into optimized calendar events.

## 1. Overview

The scheduling system is designed to automatically find the best time slots for tasks while respecting:

- Task dependencies (`blockedBy`)
- Due dates and priorities
- User working hours and schedules
- Existing calendar events (meetings and other tasks)
- Task recurrence patterns

---

## 2. The Orchestration Flow

**Core File:** [backend/src/services/autoScheduleService.ts](backend/src/services/autoScheduleService.ts)

The `AutoScheduleService` is the entry point. When a scheduling run is triggered:

1.  **Data Fetching**: It retrieves all active tasks, existing calendar events, and the user's working schedules from the database.
2.  **Horizon Enrichment**: For recurring tasks, it ensures a 90-day window is considered so future occurrences are "blocked off" even if they haven't been fully instantiated yet.
3.  **Completed-event awareness**: When expanding recurring tasks into synthetic placeholders, ALL task events (including completed ones) are passed to `expandRecurringTasks`, so that dates which already have a completed occurrence are not re-generated. Similarly, `calculateAutoSchedule` includes completed events in `lockedFutureEvents` for recurring tasks, preventing `prepareRecurringTaskEvents` from creating duplicate slots on already-completed dates.
4.  **Calculation**: It delegates the core logic to `calculateAutoSchedule`.
5.  **Diffing (`isSameSchedule`)**: It compares the _new_ proposed schedule against what is currently in the database to avoid unnecessary writes.
6.  **Atomic Update (`applyDiff`)**: If changes are needed, it performs a "wipe and replace" for the user's auto-scheduled events to ensure no duplicates or stale blocks remain.

---

## 3. The Sorting Strategy

**Core File:** [backend/src/utils/autoScheduleCalculator.ts](backend/src/utils/autoScheduleCalculator.ts)

Before finding slots, the engine must decide the order of operations.

### Topological Sort

The system first performs a topological sort based on the `blockedBy` array.

- If **Task B** depends on **Task A**, Task A is _always_ scheduled first.
- This ensures that Task B's start time is physically pushed after Task A's end time.

### Priority & Due Dates

Within the dependency tiers, tasks are sorted by:

1.  **Due Date**: Tasks expiring sooner get priority.
2.  **Priority Rank**: `high` > `medium` > `low`.
3.  **Continuation**: The `reorderTasksForContinuation` logic attempts to keep the user on the same task they were previously working on to minimize context switching.

---

## 4. The Slot Finding Logic ("The Physics Engine")

**Core File:** [backend/src/utils/taskScheduler.ts](backend/src/utils/taskScheduler.ts)

For each task in the sorted list:

### `distributeEvents`

This function attempts to fit the `planned_duration_minutes` into the calendar.

- **Starting Point**: Usually `now` (rounded to the next 15m) or the task's `start_date`.
- **Working Hours**: It checks the `Schedule` object for that day. If the user only works 9 AM - 5 PM on Mondays, it will skip Tuesday entirely.
- **Gap Management**: A default 5-minute buffer is maintained between events to prevent burnout.
- **Chunking**: If a 4-hour task only finds a 1-hour window, it schedules a 1-hour "chunk" and continues searching for the remaining 3 hours in the next available gap.

### `getNextAvailableSlot`

The low-level "searcher" that scans chronologically for a gap of at least `minBlockMinutes` (default 15) that doesn't overlap with:

- External calendar events (Google/Outlook sync)
- Already scheduled high-priority tasks
- Recurring task placeholders

**Important**: Events are **merged** before gap computation. Overlapping Google Calendar events (e.g. a duplicate train entry + a Study Time block that both cover 07:00–09:25) must be collapsed into a single interval first. Without merging, the gap finder produces phantom free slots that are actually still inside an overlapping event.

### Date-only string parsing

`task.start_date` and `task.recurrence_start_date` are stored as `YYYY-MM-DD` strings. **Never** use `new Date(dateStr)` on them — JS parses date-only strings as UTC midnight, which in UTC+1 (Italy) becomes the previous day at 23:00 local time.

Use `parseDateLocal(date)` from `taskScheduler.ts` instead:

```typescript
import { parseDateLocal } from './taskScheduler.js';
const d = parseDateLocal(task.start_date); // always local midnight
```

---

## 5. Handling Violations

If a task has a `due_date`, the system tries its best to fit it before that date. If it cannot (due to time constraints or higher priority tasks), it will **still schedule the task** as soon as possible after the deadline, but it will return it in the `violations` array to notify the user.

---

## 6. Key Data Types

- **`Task`**: The raw unit of work.
- **`CalendarEventTask`**: A specific instance (block of time) on the calendar linked to a task.
- **`Schedule`**: Defines the "valid" hours for a specific day of the week.
- **`TaskSchedulingConfig`**: Holds defaults like `gapBetweenEventsMinutes` and `minBlockMinutes`.

---

## 7. Triggering & Scheduling Lifecycle

The auto-schedule is not a continuous background process; it is **event-driven** and **debounced**.

### The Trigger Queue

**Core File:** [backend/src/services/autoScheduleTriggerQueue.ts](backend/src/services/autoScheduleTriggerQueue.ts)

To prevent "waterfall" scheduling (where multiple rapid changes cause multiple expensive re-calculations), the system uses a **500ms debounce window**.

### When is it triggered?

The `trigger()` function is called by various services whenever a mutation occurs that could affect the optimal schedule:

1.  **Task Mutations** ([TaskService](backend/src/services/taskService.ts)):
    - Creating a new task.
    - Updating a task (changing its duration, due date, priority, or status).
    - Deleting a task.
2.  **Project Mutations** ([ProjectService](backend/src/services/projectService.ts)):
    - Creating a project (which might have default due dates).
    - Updating project timelines.
3.  **Schedule Changes** ([UserSettingsService](backend/src/services/userSettingsService.ts)):
    - Modifying working hours (e.g., changing 9-5 to 10-6).
    - Toggling working days (e.g., adding Saturday as a working day).
4.  **Calendar Sync** ([GoogleCalendarService](backend/src/services/googleCalendarService.ts)):
    - When new external events are synced from Google Calendar, as these create new "busy" blocks that tasks must move around.
5.  **Manual Trigger**: Users can manually request a "Reschedule" via the `/api/auto-schedule/run` endpoint.

### Lifecycle Summary

1.  **Mutation**: User updates a task.
2.  **Queue**: `TaskService` calls `autoScheduleTriggerQueue.trigger()`.
3.  **Wait**: System waits 500ms for any other rapid changes.
4.  **Async Run**: `AutoScheduleService.run()` executes in the background (fire-and-forget).
5.  **Notify**: The schedule is updated in the database, and the frontend usually refreshes via a websocket or polling notification.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlessandroMarc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AlessandroMarc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
