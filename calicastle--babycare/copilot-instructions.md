## babycare

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # Start dev server (Vite)
pnpm build        # Type-check (tsc -b) then build for production
pnpm lint         # ESLint
pnpm preview      # Preview production build locally
```

## Tech Stack

- **React 19** + **TypeScript 5.9** (strict mode) with **Vite 7**
- **Tailwind CSS 4** via `@tailwindcss/vite` plugin (no postcss config needed)
- **React Router DOM 7** with hash-based routing (`HashRouter` in `main.tsx`)
- **@base-ui/react** — headless UI components (Tabs, Collapsible, Dialog, AlertDialog, Progress, NumberField, Toggle, ToggleGroup, ScrollArea)
- **react-day-picker** — date picker (zh-CN locale, used in bottom sheet Dialog)
- **sileo** — toast notifications (`sileo.success()`, `sileo.error()`, `sileo.info()`)
- **Dexie.js 4** (IndexedDB) for persistent data, **localStorage** for settings
- **vite-plugin-pwa** for offline-first PWA with Workbox caching
- **Nucleo icons** — `nucleo-glass` (dock nav + action button), `nucleo-ui-outline-duo-18` / `nucleo-ui-fill-duo-18` (feature tool icons)
- **pnpm** as package manager

## Architecture

This is 宝宝助手 (BabyCare) — a Chinese-only pregnancy/baby care PWA with a Duolingo-inspired UI. All UI text is hardcoded in Chinese. No backend, no auth — everything is local to the device.

### Routing

Routes are defined in `src/App.tsx`. Two kinds:

- **Layout routes** (wrapped in `<Layout>` with Dock + ScrollArea): `/`, `/history`, `/settings`, `/tools/*` home pages
- **Full-screen session routes** (no Dock, no Layout): `/tools/kick-counter/session`, `/tools/contraction-timer/session/:sessionId`, `/tools/feeding-log/session/:recordId`

When navigating away from sessions, use `navigate(path, { replace: true })` to prevent back-button confusion.

### Data Layer

- **Dexie database** (`src/lib/db.ts`): `KickCounterDB` with tables `sessions`, `contractionSessions`, `contractions`, `hospitalBagItems`, `feedingRecords`. Schema versions are incremental — always add a new `db.version(N+1)` when adding tables/indexes.
- **Settings** (`src/lib/settings.ts`): stored in localStorage under `babycare-settings`. Includes `goalCount`, `mergeWindowMinutes`, `colorMode`, `dueDate`. Also exports helpers `getDaysUntilDue()` and `getWeeksPregnant()`.

### File Organization

- `src/components/` — reusable UI components (Layout, Dock, StickyHeader, ProgressRing, TipBanner)
- `src/hooks/` — custom React hooks (useDockGesture)
- `src/lib/` — pure utilities (db, settings, time, haptics, tips, encouragements, tools, feeding-helpers, hospital-bag-presets)
- `src/pages/` — route-level components
- `src/pages/tools/<feature>/` — each tool gets its own subdirectory (kick-counter, contraction-timer, hospital-bag, feeding-log)

### Key Conventions

- Components are **default exports**: `export default function ComponentName()`
- Hooks are **named exports**: `export function useHookName()`
- Imports use **explicit `.ts`/`.tsx` extensions**
- Timestamps are **milliseconds** (`Date.now()`), IDs are **`crypto.randomUUID()`**
- No state management library — React hooks + local component state only
- Color mode via `.dark` class toggle on `<html>` — supports system/light/dark (see `applyColorMode()`)
- Use `sileo.success()` / `sileo.error()` / `sileo.info()` for user feedback instead of `alert()`

### Design System

Duolingo-inspired, flat, clean, and playful. Defined in `src/index.css` and applied via Tailwind utility classes throughout.

#### Core Principles

1. **No shadows** — Never use `shadow-*` classes on cards, containers, or buttons.
2. **Border-based contrast** — Use soft borders to define card edges: `border border-gray-200 dark:border-gray-700/60`.
3. **Bold typography** — Headlines use `font-extrabold`, labels use `font-bold`.
4. **Flat color fills** — Buttons and accents use solid Duo palette colors, never gradients (except the hero banner).

#### Color Palette (`src/index.css`)

| Token              | Hex       | Usage                                   |
|--------------------|-----------|------------------------------------------|
| `duo-green`        | `#58CC02` | Primary actions, kick counter, active toggle states |
| `duo-green-dark`   | `#46a302` | Button bottom borders (pressed look)     |
| `duo-orange`       | `#FF9600` | Streaks, contraction timer, warnings     |
| `duo-blue`         | `#1CB0F6` | Kick counter icon, informational accents |
| `duo-purple`       | `#CE82FF` | Due date, date picker accent             |
| `duo-red`          | `#FF4B4B` | Danger actions, stop buttons, alerts     |
| `duo-yellow`       | `#FFC800` | Celebrations, highlights                 |
| `duo-gray`         | `#E5E5E5` | Disabled states, separators              |

#### CSS Variables (`src/index.css`)

| Variable | Light | Dark | Usage |
|----------|-------|------|-------|
| `--sileo-fill` | `#f3f3f3` | `#161616` | Toast notification SVG fill |
| `--dock-accent-1` | `#1a1a1a` | `#ffffff` | Nucleo glass icon gradient stop 1 |
| `--dock-accent-2` | `#404040` | `#d4d4d4` | Nucleo glass icon gradient stop 2 |

#### Cards

- Background: `bg-white dark:bg-[#16213e]`
- Border: `border border-gray-200 dark:border-gray-700/60`
- Radius: `rounded-2xl` (standard) or `rounded-3xl` (hero/featured)
- Padding: `p-5` (standard) or `p-4` (compact lists)
- No shadows ever

#### Section Headers

Uppercase, tiny, bold, gray:
```
<p class="text-[11px] font-bold text-gray-400 dark:text-gray-500 uppercase tracking-wider mb-3">
  SECTION NAME
</p>
```

#### Bottom Sheet Dialog

For pickers, tool picker, and confirmations that slide up from bottom:
```tsx
<Dialog.Popup className="fixed bottom-0 left-0 right-0 bg-white dark:bg-[#16213e] rounded-t-3xl px-5 pt-5 pb-8 transition-all duration-300 data-[ending-style]:translate-y-full data-[starting-style]:translate-y-full">
  <div className="w-10 h-1 bg-gray-300 dark:bg-gray-600 rounded-full mx-auto mb-5" />
  ...
</Dialog.Popup>
```

#### Dark Mode Tokens

| Element      | Light               | Dark                   |
|-------------|----------------------|------------------------|
| Page bg     | `bg-gray-50`         | `bg-[#1a1a2e]`        |
| Card bg     | `bg-white`           | `bg-[#16213e]`        |
| Card border | `border-gray-200`    | `border-gray-700/60`  |
| Text primary| `text-gray-800`      | `text-white`           |
| Text muted  | `text-gray-400`      | `text-gray-500`        |
| Input bg    | `bg-gray-100`        | `bg-gray-800`          |

### Dock (`src/components/Dock.tsx`)

Floating bottom nav with frosted glass styling + separate action button.

**Structure**: Nav bar (left, `flex-1`) + action button (right). Container uses `justify-between px-4 gap-2`.

**Nav items**: 3 NavLinks (首页, 记录, 设置) with Nucleo glass icons + text labels. Active state uses theme-inverted colors (`text-gray-800 dark:text-white`), inactive uses `text-gray-400 dark:text-gray-500`.

**Action button**: Opens a Dialog bottom sheet with tool picker grid (shared tools from `src/lib/tools.tsx`).

**Swipe gesture** (`src/hooks/useDockGesture.ts`): Horizontal swipe across nav items shifts focus with spring scale animation (`cubic-bezier(0.34, 1.56, 0.64, 1)`) and haptic feedback. Tap behavior preserved — gesture only activates on horizontal movement >8px.

**Icon theming**: Nucleo glass icons use `--nc-gradient-1-color-1` / `--nc-gradient-1-color-2` CSS variables (set via `--dock-accent-1`/`--dock-accent-2`) for theme-aware accent gradients.

### Business Logic

- **Kick counter**: Configurable merge window (3/5/10 min) — first tap in a window increments `kickCount`, subsequent taps within the window are recorded but don't increment. Cardiff Count-to-10 method.
- **Contraction timer**: tracks duration + interval per contraction. 5-1-1 rule alert (contractions ≤5 min apart, ≥1 min long, for ≥1 hour).
- **Smart tool ordering** (`src/lib/tools.tsx`): `getWeeksPregnant()` determines tool grid order. Before 28 weeks → contraction timer first. 28+ weeks → kick counter first. Past due date → contraction timer first.
- **Hospital bag**: checklist with preset items, tracks completion.
- **Feeding log**: breast + bottle tracking with side switching and duration.

### PWA

Configured in `vite.config.ts`. Display mode is `standalone` (full-screen like native). Safe area insets handled via CSS env variables (`--safe-area-top`, `--safe-area-bottom`). Custom variant `pwa:` targets `@media (display-mode: standalone)`.

## Tool Usage

- **Always use Context7 MCP** (`resolve-library-id` → `query-docs`) when needing library/API documentation, code generation, setup or configuration steps — without the user having to explicitly ask. This ensures up-to-date docs are used instead of relying on training data.

---
> Source: [CaliCastle/babycare](https://github.com/CaliCastle/babycare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
