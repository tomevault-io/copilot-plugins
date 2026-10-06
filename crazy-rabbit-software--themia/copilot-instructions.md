## themia

> A Windows desktop widget application built with **Tauri 2** (Rust backend) and **SolidJS** (frontend). Widgets render as transparent, always-on-desktop windows using `WS_EX_NOACTIVATE` for a non-intrusive experience.

# Desktop Widget App

## Project Overview
A Windows desktop widget application built with **Tauri 2** (Rust backend) and **SolidJS** (frontend). Widgets render as transparent, always-on-desktop windows using `WS_EX_NOACTIVATE` for a non-intrusive experience.

## Tech Stack
- **Frontend**: SolidJS 1.8, TypeScript, Vite 8
- **Backend**: Rust (Tauri 2)
- **Icons**: lucide-solid (use this for all icons, never emojis in code)
- **Styling**: Plain CSS files (no CSS-in-JS, no Tailwind)

## File Structure
```
src/
  main.tsx              # Entry point
  App.tsx               # Root component (view management, design panel, widget picker)
  WidgetGrid.tsx        # Grid layout with drag/resize
  ContextMenu.tsx       # Global context menu renderer
  layout.ts             # Widget layout state (positions, sizes)
  design.ts             # Design system state (colors, gaps, corners)
  views.ts              # Multi-view state management
  ctxMenuStore.ts       # Context menu store (showCtxMenu, closeCtxMenu)
  fileClipboard.ts      # Copy/cut/paste state for file operations
  widgets/
    registry.tsx        # Widget registry: all widget types registered here

widgets/
  email/                # Email widget (Microsoft Graph + IMAP)
    Email.tsx
    EmailConfig.tsx
    Email.css
    EmailConfig.css
    src/lib.rs          # Rust backend (accounts, messages, auth)
    Cargo.toml
  folder/               # File explorer widget
    FolderView.tsx
    FolderViewConfig.tsx
    FolderView.css
    FolderViewConfig.css
  contacts/             # Contacts widget (Microsoft Graph)
    Contacts.tsx
    ContactsConfig.tsx
    Contacts.css

src-tauri/
  src/lib.rs            # Tauri command registration
```

## Design System

### CSS Variables (dark theme)
All widgets use these CSS variables; never hardcode colors:
```css
--foreground:        rgba(255, 255, 255, 0.886)   /* Primary text */
--muted-foreground:  rgba(255, 255, 255, 0.544)   /* Secondary text */
--subtle-foreground: rgba(255, 255, 255, 0.36)    /* Tertiary/preview text */
--card:              rgba(30, 30, 30, 0.90)        /* Widget background */
--card-hover:        rgba(255, 255, 255, 0.06)     /* Hover state */
--border:            rgba(255, 255, 255, 0.08)     /* Borders */
--primary:           #60cdff                       /* Accent (cyan) */
--destructive:       #ef4444                       /* Error/danger (red) */
--radius:            8px                           /* Default border radius */
--item-gap:          0px                           /* Gap between list items */
```

### Styling Conventions
- Each component has a companion `.css` file (e.g., `Email.tsx` + `Email.css`)
- Use a unique root class per component to scope styles (e.g., `.email-widget`, `.email-config-body`)
- Glass effect: `backdrop-filter: blur(60px) saturate(180%)`
- Interactive elements: `transition: background 0.12s` with `rgba(255,255,255,0.06)` hover
- Buttons: transparent background, 1px solid border with low-opacity white, `font-family: inherit`
- Scrollbars: config/settings surfaces (`.cfg`, settings modal, config window) use an always-visible styled thin scrollbar (`overflow-y: scroll` + `scrollbar-width: thin` + styled `::-webkit-scrollbar-thumb`). Widget display bodies AND dropdown/picker panels (`.dropdown-panel`, `.icon-picker-panel`) hide scrollbars (`scrollbar-width: none` + `::-webkit-scrollbar { display: none }`)
- Scroll fade: gradient overlay at bottom of scrollable widget lists

### Typography
- Font sizes: 10px (labels/badges), 11px (preview), 12px (body), 13px (controls/inputs)
- Font weights: 400 (normal), 500 (medium), 600 (semibold), 700 (bold/badges)
- Use `font-variant-numeric: tabular-nums` for dates/numbers

## Widget Architecture

### Registry Pattern (`src/widgets/registry.tsx`)
Every widget type is registered with:
```ts
{
  component: Component,           // Main display component
  configComponent: ConfigComponent, // Settings panel component
  label: string,                   // Display name
  icon: string,                    // Emoji icon for picker
  category: string,                // Grouping category
  defaultSpan: { cols, rows },     // Default grid size
  defaultConfig: {},               // Initial config values
  contextActions: (config, setConfig) => CtxMenuItem[],  // Right-click menu
}
```

### Widget Props Pattern
Display components receive config values as flat props:
```tsx
export default function MyWidget(props: MyWidgetProps) { ... }
```

Config components receive:
```tsx
interface ConfigProps {
  config: Record<string, any>;
  onChange: (config: Record<string, any>) => void;
  globals?: Record<string, any>;
}
```

### Context Menu Pattern
Two levels of context menus:
1. **Widget-level**: Defined in `registry.tsx` via `contextActions`. Triggered by right-clicking the widget chrome.
2. **Item-level**: Defined inside the component. Triggered by right-clicking individual items (emails, files, contacts).

```tsx
import { showCtxMenu } from '../../src/ctxMenuStore.ts';

function handleItemRightClick(e: MouseEvent, item: Item) {
  showCtxMenu(e, [
    { icon: SomeIcon, label: 'Action', action: () => { ... } },
    { sep: true },
    { icon: Trash2, label: 'Delete', action: () => { ... }, danger: true },
  ]);
}
```

Menu item shape: `{ icon?, label, action, sep?, header?, active?, danger?, disabled?, shortcut? }`

**IMPORTANT**: `icon` is a lucide component REFERENCE (not JSX). ContextMenu renders it via Dynamic inside its own reactive root, so passing `<Icon .../>` from an event handler leaks computations. Pass `Icon` not `<Icon />`. The renderer always uses `size=14 strokeWidth=1.5`.

### Icons
Always use **lucide-solid** for icons. Standard sizes:
- Context menu icons: pass component ref only (sizing handled by menu renderer)
- Header/inline icons: `size={13} strokeWidth={1.5}`

### Tauri Commands
Backend commands are called via `invoke()` from `@tauri-apps/api/core`. Commands are registered in `src-tauri/src/lib.rs` and implemented in widget-specific Rust crates under `widgets/*/src/lib.rs`.

## State Management
- **Signals**: `createSignal()` for local component state
- **Stores**: `createStore()` from `solid-js/store` for shared state (layout, design, views, context menu)
- **Memos**: `createMemo()` for derived/computed values
- **Effects**: `createEffect()` for side effects (data fetching, DOM updates)
- **Cleanup**: `onCleanup()` for timers, event listeners

## Build & Dev
```bash
npm run dev          # Start Vite + Tauri dev server
npm run build        # Full Tauri production build
npm run vite:build   # Frontend-only build (quick check)
npm run typecheck    # tsc --noEmit (fast, run after every change)
npm run check        # Fast gate: typecheck (+ lint/unit as they land)
```

## Engineering Principles: READ BEFORE IMPLEMENTING

This codebase had a documented history of bugs caused by a handful of repeated
habits. The hardening campaign that addressed them shipped in v0.12.0 (the detailed
review docs have since been removed; the non-negotiables below are the distilled
rules). **Do not add new instances of them.** When in doubt, prefer
correctness-by-construction over a workaround.

Non-negotiables for new/changed code:
- **No `.unwrap()`/`.expect()` in non-test Rust**, and no `.lock().unwrap()`;
  recover poisoned locks. A `#[tauri::command]` and every Win32 hook/WndProc
  callback must never panic (wrap FFI callbacks in `catch_unwind`).
- **No empty `catch {}` and no `unwrap_or_default()` on I/O.** Every error is
  handled, or logged + surfaced via the toast pipeline, or annotated
  `// ignore: <reason>`. Distinguish "absent" from "corrupt".
- **Validate at every trust boundary.** No raw `JSON.parse` into a store: parse
  through a schema with a `schemaVersion`; on failure return a typed default + a
  loud log, never a half-built object.
- **No `any` in storage/sync/config/IPC code.** Closed sets are `enum`/discriminated
  unions, not strings (provider, widget type, license tier, action, message type).
- **No new wall-clock timing in correctness paths** (no "poll until it's probably
  right", no `setTimeout` to paper over ordering). Event-driven only.
- **Persisted mutations are atomic** (temp file + rename) and single-writer; token
  refresh and any read-modify-write are single-flight. Never delete user data on a
  recoverable error (e.g. a 401 → refresh, don't remove the account).
- **RAII for every Win32/COM/GDI/clipboard handle**; free on all paths.
- **Security defaults stay secure:** never `danger_accept_invalid_certs`, never put
  secrets in source, never ship a `window.__*`/`navigator.webdriver` backdoor in a
  release build.
- **Design tokens, not literals:** use `var(--primary)` etc. and a `--z-*` scale;
  never hardcode the accent color or a magic z-index.

Process:
- Read the subsystem's `SPEC.md` (`[S-NN]` tags) before changing it. If a `SPEC.md`
  is missing, the behavior is unpinned, so add the tags + a characterization test first.
- **Separate refactor commits from behavior-change commits.** A refactor leaves all
  tests green *without editing them*. A behavior change edits the test first (red →
  green). Never mix.
- **Every bug fix ships with a regression test that failed before the fix**, named
  with the finding/spec tag (e.g. `N2_`, `S-03_`).
- **One path, properly implemented, never two.** Do not gate a structural change
  behind a flag that keeps the old implementation alive alongside the new one, and
  do not leave parallel code paths doing the same job. Replace the old path cleanly.
  For risky structural change (z-order, focus, cross-window state, persistence
  schema), de-risk with tests instead of a second path: a characterization test on
  the old behavior, the migration covered by a regression test, and an explicit
  manual multi-monitor / live pass listed in the task summary (DoD item 4), then
  cut over.

## Definition of Done (self-check before finishing ANY task)
1. `npm run check` is green (typecheck + lint + unit as configured).
2. Touched a parser / z-order / sync / fs path? Ran its characterization test and
   reviewed the golden-snapshot diff, confirming the diff is intentional.
3. Touched a user-visible feature? Ran its scoped e2e
   (`npm run test:e2e:spec -- <spec>`), or stated explicitly why it couldn't run.
4. Touched Win32 / focus / live-service code that CI can't reach? Used the `diag_*`
   commands where possible and **explicitly listed what still needs manual /
   multi-monitor / live verification** in the task summary. Never imply something
   is verified when it isn't.
5. New behavior is covered by a test; fixed bug has a regression test.

---
> Source: [crazy-rabbit-software/themia](https://github.com/crazy-rabbit-software/themia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
