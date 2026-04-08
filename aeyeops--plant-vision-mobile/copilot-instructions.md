## plant-vision-mobile

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
# Development
npm run dev          # Start Vite dev server with HMR (http://localhost:5173)

# Build
npm run build        # TypeScript compile + Vite production build

# Quality Checks
npm run lint         # Run ESLint for code quality

# Preview Production Build
npm run preview      # Preview production build locally
```

## Architecture Overview

### Core Stack
- **React 18** with TypeScript for type-safe component development
- **Vite** as build tool with SWC for fast transpilation
- **Progressive Web App (PWA)** with offline-first architecture using Workbox
- **IndexedDB** via Dexie for client-side data persistence
- **Zustand** for lightweight state management

### Key Architectural Patterns

#### Offline-First Data Flow
1. **Local Storage**: All data operations write to IndexedDB first via `src/lib/db.ts`
2. **Sync Queue**: Changes tracked in `syncQueue` table for eventual consistency
3. **Background Sync**: `SyncService` handles bidirectional sync when online
4. **Conflict Resolution**: Last-write-wins strategy for concurrent edits

#### State Management Structure
- **Global App State**: `src/stores/useAppStore.ts` - User auth, settings, sync status
- **Local Persistence**: Zustand middleware persists to localStorage
- **Database Layer**: Dexie wraps IndexedDB with reactive queries

#### Component Architecture
- **UI Components**: `src/components/ui/` - shadcn/ui base components (Button, Card, etc.)
- **Page Components**: `src/pages/` - Route-level components with lazy loading
- **Lazy Loading**: Custom retry mechanism in `src/utils/lazyLoading.tsx`
- **Error Boundaries**: Graceful error handling at component tree level

### PWA Implementation

#### Service Worker Strategy
- **Auto-update**: New versions automatically installed
- **Caching**: Static assets cached for offline use
- **Manifest**: Configured in `vite.config.ts` with PWA plugin

#### Key PWA Files
- `src/components/PWAInstallPrompt.tsx` - Install prompt handler
- `vite.config.ts` - PWA manifest and Workbox configuration

### Data Models

#### Inspection Entity (`src/lib/db.ts`)
Primary business entity with:
- UUID for cross-device sync
- Readings (temperature, pressure, flow, vibration)
- Photo attachments (base64 encoded)
- Sync status tracking

#### Sync Queue
Tracks pending operations:
- Entity reference (UUID)
- Operation type (create/update/delete)
- Snapshot of data at queue time

### Critical Services

#### SyncService (`src/services/syncService.ts`)
- Bidirectional sync with retry logic
- Batch processing (50 items per batch)
- Exponential backoff on failures
- Conflict resolution via timestamp comparison

#### Analytics & Error Tracking
- Amplitude for analytics (`src/services/analyticsService.ts`)
- Sentry for error monitoring (integrated in `App.tsx`)

### Routing & Navigation
- React Router v6 with browser router
- Bottom navigation bar (`src/components/Navigation.tsx`)
- Dynamic default view based on user settings
- Routes: Dashboard, Scanner, Viewer3D, Inspections, Settings

### 3D Visualization
- Three.js with React Three Fiber
- Located in `src/pages/Viewer3D.tsx`
- Interactive 3D models for plant inspection

### Performance Optimizations
- Route-based code splitting with retry logic
- Virtual scrolling for large lists (`src/components/VirtualList.tsx`)
- Performance monitoring hooks in `src/hooks/`
- Manual vendor chunking in Vite config

## Path Aliases
- `@/` maps to `src/` directory (configured in `vite.config.ts`)

## Security Headers
Configured in Vite dev server:
- Strict Transport Security
- X-Frame-Options: DENY
- Content Security Policy
- Camera/microphone permissions for PWA features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AeyeOps)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AeyeOps)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
