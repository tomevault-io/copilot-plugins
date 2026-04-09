## suprlend-employee-app-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Realtime workforce scheduling system for assigning operators to facility shifts. Operators interact via WhatsApp only; admins use the web dashboard. Built with Next.js (static export) on Firebase Hosting + Express.js backend server.

### Core Architecture

- **Frontend**: Next.js 16 App Router, static export (`output: "export"`), deployed on Firebase Hosting
- **Backend**: Express.js server in `/server` (port 3001) — WhatsApp, Google Forms, scheduling
- **Database**: Cloud Firestore (open rules — security via Firebase Hosting restrictions)
- **Messaging**: WhatsApp via `whatsapp-web.js` + Twilio (dual approach)
- **State Management**: Zustand for client state, Firestore real-time listeners via `useRealtimeCollection`
- **Styling**: Tailwind CSS 3

### Key Domain Concepts

- **Operator**: Worker with `certLevel` (0/1/2) and `reliabilityScore` (0-100)
- **Shift**: Time window with 8 statuses: `DRAFT` → `AVAILABILITY_OPEN` → `FINAL_CONFIRM_OPEN` → `ASSIGNMENT_IN_PROGRESS` → `PUBLISHED` → `IN_PROGRESS` → `COMPLETED` / `CANCELED`
- **OperatorShift**: Links operator to shift with 14-state machine (see [operator-shift-machine.ts](src/domain/state-machine/operator-shift-machine.ts))
- **Assignment**: Operator placed at facility or standby
- **Anchor**: Certified operator (`certLevel >= 1`) required for non-certified to enter facility
- **Settings**: Hierarchical resolution: `GLOBAL < PER_SHIFT < PER_FACILITY_PER_SHIFT`

## Common Commands

```bash
# Development
npm run dev                    # Next.js dev server (localhost:3000)
npm run dev:server             # Express backend server (localhost:3001)
npm run dev:all                # Both frontend + backend concurrently
npm run emulators              # Firebase emulators (Firestore, Auth)

# Building
npm run build                  # Build Next.js (static export to /out)
npm run build:server           # Build Express server
npm run build:all              # Build both

# Testing
npm test                       # Run Jest tests (--runInBand)
npm test -- __tests__/unit/state-machine.test.ts   # Single test file
npm run test:watch             # Watch mode
npm run test:coverage          # Coverage report

# Type checking / Linting
npm run typecheck              # TypeScript (tsc --noEmit)
npm run lint                   # ESLint (next lint)

# Data management
npm run seed:facilities        # Seed facilities collection
npm run import:operators       # Import operators from XLSX

# Deployment
npm run deploy                 # Build + deploy hosting + firestore rules
npm run deploy:web             # Hosting only
npm run deploy:rules           # Firestore rules + indexes only

# Install
npm run install:all            # Install root + server dependencies
```

## Two-Project Setup

The repo has two separate Node projects with independent `package.json` and `tsconfig.json`:

| Project | Directory | Port | Purpose |
|---------|-----------|------|---------|
| Frontend | `/` (root) | 3000 | Next.js dashboard |
| Backend | `/server` | 3001 | Express.js API server |

Install both: `npm run install:all` (or `npm install && cd server && npm install`)

## Important File Locations

| Purpose | Location |
|---------|----------|
| Domain types (single source of truth) | [src/types/domain.ts](src/types/domain.ts) |
| Assignment engine (two-pass) | [src/domain/assignment/engine.ts](src/domain/assignment/engine.ts) |
| Assignment validation rules | [src/domain/assignment/rules.ts](src/domain/assignment/rules.ts) |
| Operator-shift state machine | [src/domain/state-machine/operator-shift-machine.ts](src/domain/state-machine/operator-shift-machine.ts) |
| Settings resolution | [src/domain/settings/resolve-settings.ts](src/domain/settings/resolve-settings.ts) |
| RBAC permissions | [src/domain/rbac/permissions.ts](src/domain/rbac/permissions.ts) |
| Firestore client operations | [src/services/firestore-client-ops.ts](src/services/firestore-client-ops.ts) |
| Data access layer | [src/services/repositories.ts](src/services/repositories.ts) |
| WhatsApp parser | [src/services/whatsapp-parser.ts](src/services/whatsapp-parser.ts) |
| Firebase client init | [src/lib/firebase-client.ts](src/lib/firebase-client.ts) |
| Server entry point | [server/src/index.ts](server/src/index.ts) |
| Server WhatsApp routes | [server/src/routes/whatsapp.ts](server/src/routes/whatsapp.ts) |
| Server scheduler service | [server/src/services/scheduler-service.ts](server/src/services/scheduler-service.ts) |
| Firestore rules | [firestore.rules](firestore.rules) |
| Full PRD | [docs/PRD_SRS.md](docs/PRD_SRS.md) |
| Operational runbook | [docs/RUNBOOK.md](docs/RUNBOOK.md) |

## State Machine

Defined in [operator-shift-machine.ts](src/domain/state-machine/operator-shift-machine.ts). Use `transitionOperatorShift(currentState, event)` — returns `{ ok, nextState, reason }`.

```
NOT_INVITED → AVAILABILITY_PENDING → (AVAILABILITY_YES / NO / UNKNOWN) →
FINAL_CONFIRM_PENDING → (FINAL_YES / NO) → (ASSIGNED / STANDBY) →
(ARRIVED / LATE / NO_SHOW / CANCELED_AFTER_ASSIGNMENT)
```

Terminal states: `ARRIVED` (no transitions out), `LATE` (can → `ARRIVED`).
Recovery: `NO_SHOW` and `CANCELED_AFTER_ASSIGNMENT` can → `ASSIGNED` or `STANDBY`.

## Assignment Engine

Two-pass algorithm in [engine.ts](src/domain/assignment/engine.ts):
1. **Pass 1**: Place certified operators (anchors) round-robin across active facilities
2. **Pass 2**: Place non-certified operators only where an anchor already exists

Rules enforced via [rules.ts](src/domain/assignment/rules.ts):
- `FACILITY_FULL`: Capacity check
- `NON_CERT_REQUIRES_ANCHOR`: certLevel=0 blocked without anchor
- `MIN_CERTIFIED_BREACH`: Minimum certified count per facility
- `DUPLICATE_ASSIGNMENT`: One assignment per operator per shift
- `FACILITY_CLOSED`: Inactive facility check

Policy mode `FLEX_WITH_APPROVAL` allows rule bypass with logged reason.

## Server API Routes

Express.js server at `/server` with these route groups:

| Route | File | Purpose |
|-------|------|---------|
| `/api/whatsapp` | [routes/whatsapp.ts](server/src/routes/whatsapp.ts) | WhatsApp Web client integration |
| `/api/forms` | [routes/forms.ts](server/src/routes/forms.ts) | Google Forms API |
| `/api/messages` | [routes/messages.ts](server/src/routes/messages.ts) | Message management |
| `/api/shifts` | [routes/shifts.ts](server/src/routes/shifts.ts) | Shift operations |
| `/api/scheduler` | [routes/scheduler.ts](server/src/routes/scheduler.ts) | Auto-scheduler |
| `/health` | [index.ts](server/src/index.ts) | Health check |

CORS allows `localhost:3000` and `suprlendemployeeappweb.web.app`.

## Frontend Architecture

**Dashboard is the main UI**: [src/app/dashboard/page.tsx](src/app/dashboard/page.tsx) renders `DashboardApp` directly (AuthGate currently disabled).

**DashboardApp** ([src/components/DashboardApp.tsx](src/components/DashboardApp.tsx)) orchestrates feature panels — each is a self-contained component in its own folder under `src/components/`.

**Auth flow** ([useAuthSession.ts](src/hooks/useAuthSession.ts)):
1. Firebase Auth sign-in
2. Role resolved: custom claims → `users` collection → fallback VIEWER
3. Bootstrap admin: hardcoded email gets ADMIN role

**Real-time data** via `useRealtimeCollection` hook — wraps Firestore `onSnapshot` with `where`/`orderBy` query constraints.

## Firestore Collections

`operators`, `facilities`, `shifts`, `shiftFacilities`, `operatorShifts`, `assignments`, `messageLogs`, `auditEvents`, `settings`, `idempotency`

## Testing

Tests in `__tests__/` with three directories: `unit/`, `integration/`, `components/`.

Jest config: `ts-jest` preset, `jsdom` environment, `@/*` path alias mapped. Tests run with `--runInBand` by default.

Key test coverage: state machine transitions, assignment rules, settings resolution, phone normalization, RBAC permissions, validation schemas, WhatsApp parsing, work order export.

## Development Notes

1. Operators never access the web UI — all interaction via WhatsApp
2. Timezone hardcoded to `Asia/Jerusalem` — all dates stored as ISO/UTC, displayed in Israel time
3. RBAC is controlled by `NEXT_PUBLIC_RBAC_DISABLED` env var (defaults to enabled). AuthGate checks sessionStorage auth.
4. Next.js uses static export — no SSR/API routes, all client-side
5. `firestore.rules` has open access (`allow read, write: if true`) — security relies on hosting restrictions
6. Path alias: `@/*` → `src/*`
7. `tsconfig.json` excludes `server/`, `scripts/`, `__tests__/` from compilation
8. Reliability score default is 50
9. Violation messages in the assignment engine are in Hebrew

## Environment Setup

Copy `.env.example` to `.env.local` for Firebase client config (`NEXT_PUBLIC_*` vars). Server environment: copy `server/.env.example` for Twilio, Firebase Admin, and Google API credentials.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sagi11160)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sagi11160)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
