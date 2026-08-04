## quartermaster

> Quartermaster is a **modular gaming toolkit hub** -- a desktop app that serves as a

# Quartermaster - Project Instructions

## Project Overview

Quartermaster is a **modular gaming toolkit hub** -- a desktop app that serves as a
central dashboard for game-specific tools, databases, and utilities. Think of it like
a workbench where each drawer is dedicated to a different game.

**Core ideas:**
- **Offline-first**: The app saves a local snapshot of all data. If the internet drops,
  everything keeps working. When connectivity returns, it syncs only what changed.
- **Modular**: Each game is a self-contained plugin. You can enable or disable them
  independently. Adding a new game never breaks existing ones.
- **Desktop-first**: Ships as a native desktop app via Tauri. Mobile support comes later
  using the same Tauri v2 codebase.
- **Dark theme by default**: Because we are not animals.
- **Personal use first**: Build for yourself, polish for public release later.

**Target games (in priority order):**
1. Escape from Tarkov -- first module, serves as the testbed for the plugin system
2. Elite Dangerous
3. Star Citizen

---

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Desktop shell | **Tauri v2** | Lightweight native wrapper, Rust backend, future mobile support |
| Frontend | **React 19 + TypeScript** | Component-based UI, strong typing catches bugs early |
| Styling | **Tailwind CSS v4** | Utility-first, easy dark theme, fast iteration |
| State management | **Zustand** | Simple, minimal boilerplate, works great with React |
| Local DB (browser) | **Dexie.js (IndexedDB)** | Offline snapshot storage, fast reads from the browser side |
| Local DB (native) | **SQLite via Tauri plugin** | Persistent storage on disk, survives app reinstalls |
| API client | **urql or graphql-request** | Lightweight GraphQL client for Tarkov.dev API |
| Build tool | **Vite** | Fast dev server, comes with Tauri scaffolding |

### Why two databases?

This is a common pattern in offline-first apps. Think of it this way:

- **Dexie/IndexedDB** lives inside the browser/webview. It is fast for the UI to read
  from and is where the "current working data" lives. The React app talks directly to it.
- **SQLite** lives on the native filesystem via Tauri's Rust backend. It is the durable
  "source of truth" that persists even if the webview cache gets cleared. It also handles
  things the browser cannot do, like file exports or background sync.

Data flows: `API --> SQLite (persist) --> Dexie (UI cache) --> React components`

---

## Architecture Overview

### The Big Picture

```
+------------------------------------------------------------------+
|                        QUARTERMASTER APP                          |
|                                                                   |
|  +------------------+   +-------------------------------------+  |
|  |                  |   |           GAME MODULES              |  |
|  |    HUB CORE      |   |                                     |  |
|  |                  |   |  +----------+  +---------+  +-----+ |  |
|  |  - Shell / Nav   |   |  | Tarkov   |  | Elite   |  | SC  | |  |
|  |  - Theme engine  |   |  | Module   |  | Module  |  | Mod | |  |
|  |  - Plugin loader |   |  |          |  |         |  |     | |  |
|  |  - Sync engine   |   |  +----------+  +---------+  +-----+ |  |
|  |  - Settings      |   |                                     |  |
|  |  - Offline mgr   |   +-------------------------------------+  |
|  |                  |                                             |
|  +------------------+                                             |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |                    SHARED SERVICES                          |  |
|  |  Data layer (Dexie + SQLite) | API client | Network monitor |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |                    TAURI BACKEND (Rust)                     |  |
|  |  SQLite plugin | File system | System tray | Auto-updater  |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

**How to read this:**

1. **Hub Core** is the app shell -- navigation, settings, theme, the plugin loader that
   mounts/unmounts game modules. It exists regardless of which games are enabled.
2. **Game Modules** are independent plugins. Each one registers itself with the hub and
   gets a navigation entry. Disabling a module removes it from the UI entirely.
3. **Shared Services** are utilities that any module can use: the database layer, the
   network-aware API client, the offline/online detection.
4. **Tauri Backend** is the Rust layer that provides native OS capabilities.

### What is a "module"?

A module is a folder that exports a standard shape:

```typescript
// Every game module exports this contract
export interface GameModule {
  id: string;              // "tarkov", "elite-dangerous", "star-citizen"
  name: string;            // Display name
  version: string;         // Semver
  icon: React.ComponentType; // Sidebar icon
  routes: RouteConfig[];   // Pages this module adds
  onEnable?: () => void;   // Called when user enables the module
  onDisable?: () => void;  // Called when user disables the module
}
```

The hub core imports these, checks which ones the user has enabled in settings, and
renders only the active ones. This means you can build each game module in isolation
without worrying about the others.

---

## Project Structure

```
quartermaster/
|
|-- src-tauri/                  # Tauri / Rust backend
|   |-- src/
|   |   |-- main.rs             # Tauri entry point
|   |   |-- commands/           # Rust commands exposed to frontend
|   |   |   |-- sync.rs         # Background sync logic
|   |   |   |-- database.rs     # SQLite operations
|   |   |-- lib.rs
|   |-- tauri.conf.json         # Tauri configuration
|   |-- Cargo.toml
|
|-- src/                        # React frontend
|   |-- main.tsx                # App entry point
|   |-- App.tsx                 # Root component, mounts hub + modules
|   |
|   |-- core/                   # HUB CORE - always loaded
|   |   |-- shell/              # App shell (layout, sidebar, titlebar)
|   |   |   |-- AppShell.tsx
|   |   |   |-- Sidebar.tsx
|   |   |   |-- Titlebar.tsx
|   |   |-- theme/              # Dark/light theme engine
|   |   |   |-- ThemeProvider.tsx
|   |   |   |-- tokens.css      # Design tokens (colors, spacing)
|   |   |-- plugins/            # Module loader system
|   |   |   |-- PluginLoader.tsx
|   |   |   |-- registry.ts    # Lists all available modules
|   |   |-- settings/           # User preferences
|   |   |   |-- SettingsPage.tsx
|   |   |   |-- settingsStore.ts  # Zustand store for settings
|   |   |-- router.tsx          # App routing setup
|   |
|   |-- modules/                # GAME MODULES - each is self-contained
|   |   |-- tarkov/             # Escape from Tarkov module
|   |   |   |-- index.ts        # Module definition (exports GameModule)
|   |   |   |-- TarkovDashboard.tsx
|   |   |   |-- pages/          # Module-specific pages
|   |   |   |   |-- ItemsPage.tsx
|   |   |   |   |-- AmmoPage.tsx
|   |   |   |   |-- MapsPage.tsx
|   |   |   |   |-- TasksPage.tsx
|   |   |   |-- components/     # Module-specific UI components
|   |   |   |   |-- ItemCard.tsx
|   |   |   |   |-- AmmoTable.tsx
|   |   |   |   |-- PriceChart.tsx
|   |   |   |-- data/           # Module data layer
|   |   |   |   |-- tarkovApi.ts      # GraphQL queries for tarkov.dev
|   |   |   |   |-- tarkovDb.ts       # Dexie table definitions
|   |   |   |   |-- tarkovSync.ts     # Sync logic for this module
|   |   |   |   |-- fallback/         # Bundled fallback JSON data
|   |   |   |-- stores/         # Module-specific Zustand stores
|   |   |   |   |-- tarkovStore.ts
|   |   |   |-- types/          # TypeScript types for Tarkov data
|   |   |       |-- items.ts
|   |   |       |-- ammo.ts
|   |   |
|   |   |-- elite-dangerous/    # (future) same structure as tarkov/
|   |   |-- star-citizen/       # (future) same structure as tarkov/
|   |
|   |-- shared/                 # SHARED SERVICES - used by any module
|   |   |-- db/                 # Database layer
|   |   |   |-- dexieInstance.ts     # Dexie database setup
|   |   |   |-- sqliteBridge.ts      # Tauri SQLite commands wrapper
|   |   |-- api/                # API utilities
|   |   |   |-- graphqlClient.ts     # Configured GraphQL client
|   |   |   |-- networkMonitor.ts    # Online/offline detection
|   |   |-- sync/               # Sync engine
|   |   |   |-- SyncManager.ts       # Orchestrates online/offline sync
|   |   |   |-- SyncStatus.tsx       # UI indicator component
|   |   |-- hooks/              # Shared React hooks
|   |   |   |-- useOnlineStatus.ts
|   |   |   |-- useSync.ts
|   |   |-- ui/                 # Shared UI components (design system)
|   |   |   |-- Button.tsx
|   |   |   |-- Card.tsx
|   |   |   |-- Table.tsx
|   |   |   |-- SearchInput.tsx
|   |   |   |-- LoadingSpinner.tsx
|   |   |-- utils/              # General utilities
|   |       |-- formatters.ts        # Number/currency formatting
|   |       |-- time.ts              # Time helpers
|   |
|   |-- types/                  # Global TypeScript types
|       |-- module.ts           # GameModule interface
|       |-- sync.ts             # Sync-related types
|       |-- global.d.ts         # Global type declarations
|
|-- public/                     # Static assets
|   |-- icons/                  # Game icons, app icon
|
|-- tailwind.config.ts          # Tailwind configuration
|-- tsconfig.json
|-- package.json
|-- vite.config.ts
|-- CLAUDE.md                   # This file
```

### Key conventions

- **Each module is a folder under `src/modules/`** with the same internal structure.
  Copy the tarkov folder as a template when adding a new game.
- **Modules never import from other modules.** They only import from `shared/` or
  their own folder. This keeps them independent.
- **Shared components go in `src/shared/ui/`.** If you build a component for Tarkov
  that would be useful for other games, move it there.
- **One Zustand store per concern.** Settings get their own store, each module gets
  its own store. Do not put everything in one giant store.

---

## Core Modules Breakdown

### Hub Core (always loaded)

| Component | Responsibility |
|-----------|---------------|
| **AppShell** | Main layout -- sidebar, content area, titlebar |
| **PluginLoader** | Reads enabled modules from settings, lazy-loads them, mounts their routes |
| **ThemeProvider** | Manages dark/light theme (dark by default), exposes CSS tokens |
| **Settings** | User preferences: enabled modules, theme, sync interval, data preferences |
| **Router** | Maps URLs to pages. Each module registers its own routes through the plugin system |

### Shared Services (used by all modules)

| Service | Responsibility |
|---------|---------------|
| **Dexie instance** | Single IndexedDB database. Each module defines its own tables within it |
| **SQLite bridge** | Wrapper around Tauri SQLite commands. Handles the persistent copy of data |
| **GraphQL client** | Pre-configured client with auth headers, error handling, caching |
| **Network monitor** | Detects online/offline state, emits events that the sync engine listens to |
| **Sync manager** | Coordinates fetching fresh data when online, serving cached data when offline |
| **Shared UI kit** | Buttons, cards, tables, inputs -- consistent look across all modules |

### Tarkov Module (first game, testbed)

| Feature | Description |
|---------|-------------|
| **Items database** | Searchable/filterable list of all in-game items with prices, slots, stats |
| **Ammo charts** | Penetration vs damage charts, armor class comparisons |
| **Maps reference** | Interactive or static map images with key locations |
| **Task tracker** | Quest/task checklist with required items highlighted |
| **Flea market prices** | Current trader and flea market pricing (from tarkov.dev) |

The Tarkov module is the testbed. Every pattern you establish here (data fetching, caching,
UI components, sync logic) becomes the template for future modules.

---

## Data Flow

### Online Mode (happy path)

```
User opens app
      |
      v
[Network Monitor] --> detects: ONLINE
      |
      v
[Sync Manager] --> "Time to fetch fresh data?"
      |              (checks: last sync timestamp vs configured interval)
      |
      +-- YES --> [GraphQL Client] ---> tarkov.dev API
      |                |
      |                v
      |           API returns fresh data
      |                |
      |                v
      |           [SQLite] <-- persist full dataset (source of truth)
      |                |
      |                v
      |           [Dexie/IndexedDB] <-- update UI cache
      |                |
      |                v
      |           [React Components] <-- re-render with fresh data
      |
      +-- NO  --> [Dexie/IndexedDB] --> serve cached data --> [React Components]
```

### Offline Mode (no internet)

```
User opens app
      |
      v
[Network Monitor] --> detects: OFFLINE
      |
      v
[Sync Manager] --> skip API calls, go straight to cache
      |
      v
[Dexie/IndexedDB] --> serve last known snapshot --> [React Components]
      |
      (App works normally, user sees "Offline" badge in UI)
      |
      v
[Network Monitor] --> detects: BACK ONLINE
      |
      v
[Sync Manager] --> "What changed since last sync?"
      |
      v
[GraphQL Client] --> fetch only updated data (delta sync)
      |
      v
[SQLite] --> merge updates --> [Dexie] --> [React Components refresh]
```

### Key concepts explained

**Snapshot**: A full copy of the data at a point in time. When you first launch the app
online, it downloads everything and saves it locally. That local copy is the snapshot.

**Delta sync**: Instead of re-downloading everything, the app asks "what changed since
my last sync?" and only fetches the differences. This is faster and uses less bandwidth.
The tarkov.dev API supports this through its `updated` timestamp fields.

**Two-tier storage**: Dexie is fast but lives in the browser sandbox. SQLite lives on
disk and survives cache clears. On app launch, SQLite hydrates Dexie, then Dexie serves
the UI. Think of SQLite as your filing cabinet and Dexie as your desk.

---

## Development Roadmap

### Phase 0 -- Scaffolding (Week 1)
> Get the app running with nothing in it. Prove the tools work together.

- [ ] Initialize Tauri v2 + React + TypeScript project
- [ ] Configure Tailwind CSS with dark theme as default
- [ ] Set up the AppShell (sidebar + content area + custom titlebar)
- [ ] Create the ThemeProvider with dark/light toggle
- [ ] Install and configure Zustand with a simple settings store
- [ ] Verify the app builds and runs on desktop
- [ ] Set up the project in version control (git init, .gitignore)

**You are done when:** A dark-themed desktop window opens with a sidebar and empty content area.

### Phase 1 -- Plugin System + Offline Foundation (Week 2-3)
> Build the skeleton that all game modules will plug into.

- [ ] Define the `GameModule` TypeScript interface
- [ ] Build the PluginLoader that reads enabled modules and mounts their routes
- [ ] Create the module registry (static list for now, no dynamic loading needed)
- [ ] Set up Dexie with a simple test table
- [ ] Set up SQLite via Tauri plugin with a simple test table
- [ ] Build the SQLite-to-Dexie hydration flow (on app launch)
- [ ] Build the NetworkMonitor hook (`useOnlineStatus`)
- [ ] Create the SyncManager skeleton (online/offline branching logic)
- [ ] Add an "Offline" / "Online" status badge to the shell
- [ ] Build the Settings page (enable/disable modules, sync interval)

**You are done when:** You can toggle a dummy module on/off, and the app correctly
detects online/offline state.

### Phase 2 -- Tarkov Module: Data Layer (Week 3-5)
> Connect to real data. Get items loading from tarkov.dev.

- [ ] Set up the GraphQL client (urql or graphql-request)
- [ ] Write GraphQL queries for tarkov.dev (items, ammo, barters, tasks)
- [ ] Define TypeScript types for Tarkov data structures
- [ ] Create Dexie tables for Tarkov data (items, ammo, quests)
- [ ] Create corresponding SQLite tables
- [ ] Implement the Tarkov sync flow: API --> SQLite --> Dexie
- [ ] Bundle fallback JSON data for first-launch offline scenario
- [ ] Implement delta sync using tarkov.dev's `updated` field
- [ ] Test: disconnect internet, verify app still shows cached data
- [ ] Test: reconnect, verify sync picks up changes

**You are done when:** Tarkov item data loads from the API, persists locally, and
survives going offline.

### Phase 3 -- Tarkov Module: UI (Week 5-8)
> Build the actual user-facing features.

- [ ] Tarkov dashboard page (overview / quick stats)
- [ ] Items database page with search and filters
- [ ] Item detail view (card or modal with full stats, price, barter info)
- [ ] Ammo comparison page (table with pen/damage, sortable)
- [ ] Ammo chart visualization (pen vs damage scatter or bar chart)
- [ ] Maps reference page (static images with labels to start)
- [ ] Task tracker page (quest list with checkbox progress)
- [ ] Build shared UI components as you go (Table, Card, SearchInput, etc.)
- [ ] Move any reusable components from `modules/tarkov/components/` to `shared/ui/`

**You are done when:** You have a functional Tarkov toolkit that you actually want to
use while playing the game.

### Phase 4 -- Polish and Robustness (Week 8-10)
> Make it feel solid.

- [ ] Error handling: graceful API failures, user-friendly error messages
- [ ] Loading states: skeleton screens or spinners during data fetches
- [ ] Sync conflict resolution (last-write-wins is fine for read-only data)
- [ ] Keyboard shortcuts for common actions
- [ ] Responsive layout tweaks (works well at different window sizes)
- [ ] Performance audit: are large item lists rendering fast? (virtualization if needed)
- [ ] Settings persistence (save to SQLite, restore on launch)
- [ ] System tray support (minimize to tray)
- [ ] Auto-updater setup via Tauri

**You are done when:** The app feels reliable and pleasant to use daily.

### Phase 5 -- Second Module: Elite Dangerous (Week 10+)
> Validate the plugin architecture by adding a second game.

- [ ] Copy the Tarkov module folder structure as a template
- [ ] Identify Elite Dangerous data sources (EDDB, Inara, EDSM APIs)
- [ ] Define data types, tables, and sync logic for Elite data
- [ ] Build Elite-specific pages (ship outfitting, trade routes, exploration)
- [ ] Confirm that enabling/disabling modules works cleanly with two modules
- [ ] Extract any more shared components discovered during this phase

### Phase 6 -- Future
- Star Citizen module
- Mobile build via Tauri v2 mobile targets
- Public release prep (installer, docs, settings export/import)
- Optional: user accounts and cloud sync (only if needed)

---

## Development Guidelines

### Code style
- Use TypeScript strict mode. No `any` types unless absolutely necessary.
- Functional React components only. No class components.
- Name files in PascalCase for components (`ItemCard.tsx`), camelCase for utilities
  (`formatters.ts`).
- One component per file. Keep files under 200 lines; split if larger.

### State management
- **Zustand stores** for app-level state (settings, sync status, module state).
- **React state** (`useState`, `useReducer`) for local component state.
- **Dexie live queries** (`useLiveQuery`) for reactive database reads in the UI.
- Never put API response data directly into Zustand. It goes: API --> DB --> UI.

### Module rules
- Modules must not import from other modules. Only from `shared/` or themselves.
- Each module must export a `GameModule` object from its `index.ts`.
- Module-specific types go in the module's `types/` folder, not the global `types/`.
- Module stores must be namespaced (e.g., `useTarkovStore`, not `useStore`).

### Styling
- Use Tailwind utility classes for all styling.
- Define design tokens (colors, spacing, border radius) in `core/theme/tokens.css`.
- Dark theme is the default. Light theme is a nice-to-have, not a priority.
- Use CSS custom properties for theme values so Tailwind classes adapt automatically.

### Data flow pattern
```
API  --(fetch)-->  SQLite  --(hydrate)-->  Dexie  --(useLiveQuery)-->  Component
```
Always follow this flow. Components never call APIs directly.

### Error handling
- Wrap API calls in try/catch. On failure, fall back to cached data.
- Show user-friendly error messages, not raw error objects.
- Log errors to console in dev, and optionally to a local error log in production.

### Git workflow
- Commit often with descriptive messages.
- Use feature branches for new modules or major features.
- Main branch should always build and run.

---

## Build & Run

```bash
# Install dependencies
npm install

# Start development (Tauri + Vite dev server)
npm run tauri dev

# Build for production
npm run tauri build

# Run frontend only (no Tauri, for quick UI iteration)
npm run dev

# Type check
npm run typecheck

# Lint
npm run lint
```

### Prerequisites
- Node.js 20+
- Rust toolchain (rustup) -- required by Tauri
- Tauri CLI: `npm install -g @tauri-apps/cli`

---

## Key APIs and References

| Resource | URL | Notes |
|----------|-----|-------|
| Tarkov.dev API | https://api.tarkov.dev/graphql | GraphQL, no auth needed, rate-limited |
| Tarkov.dev Docs | https://tarkov.dev/api/ | Schema explorer and examples |
| Tauri v2 Docs | https://v2.tauri.app/ | Desktop + mobile guide |
| Dexie.js Docs | https://dexie.org/ | IndexedDB wrapper |
| Zustand Docs | https://zustand-demo.pmnd.rs/ | State management |
| Tailwind Docs | https://tailwindcss.com/docs | Utility CSS reference |

---

## Notes
- Project created: 2026-03-27
- Owner: Alexei Rosetti (Baron)
- Architecture plan version: 1.0
- This is a learning project. Patterns may evolve as understanding deepens.
  Update this document when major decisions change.

---
> Source: [DrBaron420/Quartermaster](https://github.com/DrBaron420/Quartermaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
