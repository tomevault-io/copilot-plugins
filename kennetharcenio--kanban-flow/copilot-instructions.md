## kanban-flow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm start` — dev server at http://localhost:4200/
- `npm run build` — production build (output: `dist/`, baseHref: `/kanban-flow/` for GitHub Pages)
- `npm test` — run all unit tests (Vitest via `ng test`)
- `npm run watch` — rebuild on file changes (dev config)

## Architecture

Angular 21 application — a collaborative kanban board with event sourcing, Google Drive storage, and offline support.

**All components are standalone** (no NgModules). Routes are lazy-loaded.

### Routing (`src/app/app.routes.ts`)

- `/boards` → `BoardListComponent` (board creation and listing)
- `/boards/:id` → `BoardComponent` (kanban board with drag-drop)
- `/` and `**` redirect to `/boards`

### Core Services (`src/app/core/`)

**Event Sourcing** (`event-store/`):
- `EventStoreService` — dispatches events, maintains board/cards state as BehaviorSubjects, tracks pending events for sync
- `EventReplayService` — pure deterministic replay: `replay(initialState, events) => BoardState`
- `SnapshotService` — periodic snapshots (every 50 events) to avoid full replay

**Sync** (`sync/`):
- `SyncService` — polling-based bi-directional sync (30s prod, 10s dev), offline queue via localStorage, conflict detection via revision IDs

**Auth** (`auth/`):
- `AuthService` — Google OAuth via GIS SDK, signals-based state (`isAuthenticated`, `accessToken`, `user`), dev bypass mode

**Drive** (`drive/`):
- `DriveService` — Google Drive REST API v3 wrapper for JSON file CRUD
- `LocalDriveService` — localStorage mock for dev/offline use

### State Flow

User action → `EventStoreService.dispatch()` → event stored + state replayed → `SyncService` pushes pending events to Drive → polling pulls remote events → `mergeRemoteEvents()` replays into current state.

### localStorage Keys

`kanbanflow_board_{id}`, `kanbanflow_events_{id}`, `kanbanflow_boards`, `kanbanflow_event_queue`, `kanbanflow_folder_id`

### Shared (`src/app/shared/`)

- `models/` — TypeScript interfaces: Board, Card, Column, Member, BoardEvent, BoardSnapshot, AppIndex
- `components/` — confirm-dialog, member-avatar, label-picker

## Code Style

- SCSS for styles, Angular Material dark theme (violet primary, blue tertiary)
- Prettier: 100 char width, single quotes, angular HTML parser (`.prettierrc`)
- 2-space indentation (`.editorconfig`)
- TypeScript strict mode with `noImplicitOverride`, `noImplicitReturns`, `noFallthroughCasesInSwitch`
- OnPush change detection compatible; uses Angular signals where appropriate

## Environment Config

- `src/environments/environment.ts` — production settings
- `src/environments/environment.development.ts` — dev settings (`devBypassAuth: true`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kennetharcenio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kennetharcenio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
