## spart-board

> **SPART Board** is an interactive, widget-based classroom management application built with **React 19**, **TypeScript**, and **Vite**. It leverages **Firebase** for backend services (Authentication, Firestore, Storage) and features a drag-and-drop interface for various classroom tools (timers, polls, noise meters, etc.). The development environment includes access to the **Firebase CLI** for rule and index deployment.

# Gemini Context: SPART Board

## Project Overview

**SPART Board** is an interactive, widget-based classroom management application built with **React 19**, **TypeScript**, and **Vite**. It leverages **Firebase** for backend services (Authentication, Firestore, Storage) and features a drag-and-drop interface for various classroom tools (timers, polls, noise meters, etc.). The development environment includes access to the **Firebase CLI** for rule and index deployment.

**Key Technologies:**

- **Frontend:** React 19, TypeScript, Vite, Tailwind CSS (Custom Brand Theme), Lucide React
- **Backend:** Firebase (Auth, Firestore, Storage)
- **AI:** Gemini API (via `@google/genai`)
- **Package Manager:** pnpm
- **Testing:** Vitest (Unit), Playwright (E2E), Istanbul (Coverage)

## Project Structure & Architecture

**CRITICAL NOTE:** This project uses a **flat-ish file structure**. Most source code resides at the project root (e.g., `App.tsx`, `types.ts`, `index.tsx`), but specialized logic is organized into directories like `/components/`, `/context/`, `/hooks/`, `/utils/`, and `/config/`. There is **NO `src/` directory** for primary application code (any existing `src/` is for temporary scratch files or isolated testing).

### Directory Layout

- `@/` (Root Alias): Maps to the project root directory.
- `/components/`: React components.
  - `admin/`: Admin-specific components (`AdminSettings.tsx`, `FeaturePermissionsManager.tsx`).
  - `auth/`: Authentication UI (`LoginScreen.tsx`).
  - `common/`: Reusable components (e.g., `DraggableWindow.tsx`, `Button.tsx`).
  - `layout/`: Core layout (`Sidebar.tsx`, `Dock.tsx`, `DashboardView.tsx`).
  - `widgets/`: Individual widget implementations and `WidgetRenderer.tsx`.
    - `random/`: Specialized sub-folder for the refactored Random widget.
    - `MaterialsWidget/`: Specialized sub-folder for the Materials widget.
- `/context/`: React Context providers (`AuthContext`, `DashboardContext`).
- `/hooks/`: Custom hooks (`useFirestore`, `useStorage`).
- `/config/`: Configuration files (`firebase.ts`, `tools.ts`, `widgetGradeLevels.ts`).
- `/utils/`: Shared utility functions.
- `/functions/`: Firebase Cloud Functions (backend logic).
- `/tests/`: Test setup and E2E tests.
  - `e2e/`: Playwright end-to-end tests.
- `/types.ts`: Global TypeScript definitions.
- `/App.tsx`: Root application component.
- `/index.tsx`: Entry point.
- `/Dockerfile` & `/nginx.conf`: Infrastructure for containerized deployment.

### Architecture Patterns

- **Lazy Loading:** To optimize initial bundle size, all widgets and their settings panels are lazily loaded via `React.lazy` and managed in `components/widgets/WidgetRegistry.ts`.
- **State Management:** Centralized via React Context (`DashboardContext`, `AuthContext`).
- **Widget System:** Plugin-based. New widgets are added to `components/widgets/`, registered in `types.ts` (`WidgetType`) and `config/tools.ts` (`TOOLS`), and mapped in `WidgetRenderer.tsx`.
- **Grade Level Filtering:** Widgets are categorized by grade levels (K-2, 3-5, 6-8, 9-12) in `config/widgetGradeLevels.ts`.
- **Feature Permissions:** Access to widgets can be toggled and restricted to admins or beta users via the `FeaturePermission` system and managed in `AdminSettings`.
- **Weather Logic:** Admins can configure custom temperature ranges (Above, Below, or Range) in `FeaturePermissionsManager.tsx`, which the `WeatherWidget` uses to display conditional messages and images.
- **CORS Proxy Logic:** Certain widgets (e.g., `WeatherWidget`, `LunchCountWidget`) use a multi-proxy fallback mechanism to bypass cross-origin restrictions for third-party APIs.
- **Data Persistence:**
  - **Primary:** Firestore (real-time sync).
  - **Fallback:** `localStorage` (migrated to Firestore on sign-in).
- **Security:** Firestore Security Rules enforce ownership (users can only edit their own dashboards) and admin privileges.

## Design System & Styling

- **Typography**: **Lexend** (UI), **Patrick Hand** (Accents), **Roboto Mono** (Code).
- **Brand Colors**: Custom brand colors configured in `tailwind.config.js`:
  - **Brand Blue**: Primary (#2d3f89), Dark (#1d2a5d), Light (#4356a0), Lighter (#eaecf5).
  - **Brand Red**: Primary (#ad2122), Dark (#7a1718), Light (#c13435), Lighter (#e5c7c7).
  - **Brand Gray**: A semantic grayscale palette (darkest to lightest) for UI consistency.
- **Theme Config**: See `tailwind.config.js` for full palette and extensions.

## Slash Commands

The following slash commands are available to streamline the development workflow:

- **/clean**: Deletes all unsaved work to return to the last saved state.
  - _Action:_ `git reset --hard HEAD` and `git clean -fd`.
  - _Caution:_ Permanently deletes changes since the last `/preview`.
- **/preview**: Saves your current work and updates your online preview.
  - _Action:_ Stages, commits, and pushes changes to the current branch.
- **/submit**: Integrates latest changes from `main`, resolves conflicts, verifies with tests, pushes to remote, and submits your work for review.
  - _Action:_ Fetches `main`, rebases, runs `pnpm run validate`, pushes to remote, then creates or updates a Pull Request (PR).
- **/sync**: Updates your workspace with the latest changes from the main project and syncs with your remote branch.
  - _Action:_ Fetches `origin/main`, rebases current branch, handles stashing, and pushes to remote.
- **/undo**: Reverses your last 'save' while keeping your work in the editor.
  - _Action:_ `git reset --soft HEAD~1`.

## Development Workflow

### Key Commands

- **Install Dependencies:** `pnpm install`
- **Start Dev Server:** `pnpm run dev` (Port 3000)
- **Build for Production:** `pnpm run build`
- **Preview Build:** `pnpm run preview`
- **Linting:** `pnpm run lint` (Zero-tolerance policy)
- **Fix Linting:** `pnpm run lint:fix`
- **Type Check:** `pnpm run type-check`
- **Format:** `pnpm run format`
- **Run All Checks:** `pnpm run validate`
- **Unit Tests:** `pnpm test`
- **E2E Tests:** `pnpm run test:e2e`
- **Coverage:** `pnpm run test:coverage`
- **Firebase CLI:** `pnpm firebase [command]` (e.g., `pnpm firebase deploy --only storage`)

### Firebase Management

The agent has access to the Firebase CLI via `pnpm firebase`. This allows for deploying security rules (`firestore.rules`, `storage.rules`) and indexes directly from the environment.

### Code Quality & Standards

- **Strict TypeScript:** No implicit `any`. Explicitly annotate types.
- **ESLint/Prettier:** Code must be linted and formatted. Pre-commit hooks (Husky) enforce this.
- **Path Aliases:** Always use `@/` to import from the root.
  - _Correct:_ `import { useAuth } from '@/context/useAuth'`
  - _Incorrect:_ `import { useAuth } from '../../context/useAuth'` or `import ... from 'src/context/...'`

### React useEffect Guidelines

`useEffect` is an escape hatch for syncing with **external systems** only. Do not use it as a general-purpose reactive tool.

**✅ Legitimate uses** (external systems): Firestore/Firebase listeners, DOM event listeners, timers/intervals, Web Audio API, browser APIs (`localStorage`, `navigator`, `IntersectionObserver`), Google Drive, third-party SDKs.

**❌ Avoid useEffect for:**

- **Derived state**: Compute inline during render. `const total = items.reduce(...)` — no state, no effect.
- **Expensive derivations**: Use `useMemo` instead.
- **Ref synchronization**: Assign `myRef.current = value` directly in the render body.
- **Resetting state on prop changes**: Pass a `key` prop to reset the entire subtree, or use the "adjusting state while rendering" pattern.
- **Chained state updates**: Consolidate into the triggering event handler.

**"Adjusting state while rendering" pattern** (replaces a reactive effect that resets derived state):

```tsx
const [prevItems, setPrevItems] = useState(items);
if (items !== prevItems) {
  setPrevItems(items);
  setSelection(null); // React re-renders immediately; stops when prevItems === items
}
```

## Widget Development Guide

To add a new widget:

1.  **Define Type:** Add the new type string to `WidgetType` in `types.ts`.
2.  **Register Metadata:** Add a new entry to the `TOOLS` array in `config/tools.ts` (icon, label, color).
3.  **Create Component:** Build the widget in `components/widgets/YourWidget.tsx`.
    - Must accept `widget: WidgetData` prop.
    - Use `useDashboard()` for state updates.
    - Use `WidgetLayout` component for standard header/content/footer structure.
4.  **Register Renderer:** Add the component to the switch statement in `components/widgets/WidgetRenderer.tsx`.
5.  **Set Defaults:** Define default dimensions and config in `context/DashboardContext.tsx` (`addWidget` function).

### Widget Content Scaling (CRITICAL)

Widgets with `skipScaling: true` in `WidgetRegistry.ts` use **CSS Container Queries** for responsive sizing. All text, icons, spacing, and sizing in widget **front-face content** must use container query units via inline `style={{}}` props.

```tsx
// CORRECT - scales with widget size
<span style={{ fontSize: 'min(14px, 3.5cqmin)' }}>Label</span>
<Icon style={{ width: 'min(24px, 6cqmin)', height: 'min(24px, 6cqmin)' }} />
<div style={{ padding: 'min(16px, 3cqmin)', gap: 'min(12px, 2.5cqmin)' }}>

// WRONG - fixed sizes, won't scale when widget is resized
<span className="text-sm">Label</span>
<Icon className="w-12 h-12" />
<Icon size={24} />
```

**Key rules:**

- **Never** use hardcoded Tailwind text/icon size classes (`text-sm`, `text-xs`, `w-12 h-12`, `size={24}`) in widget content.
- **Settings panels (back-face):** Normal Tailwind classes are fine, no scaling needed.
- **Empty/error states:** Use the shared `ScaledEmptyState` component (`components/common/ScaledEmptyState.tsx`) instead of hand-rolling per-widget empty states.
- **Container query units:** `cqw` = 1% container width, `cqh` = 1% container height, `cqmin` = 1% of the smaller dimension.
- **Reference implementations:** `ClockWidget.tsx`, `WeatherWidget.tsx`, `PollWidget.tsx`.

## Environment Configuration

Required variables in `.env.local` (do not commit):

```env
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
VITE_GEMINI_API_KEY=...
VITE_OPENWEATHER_API_KEY=...
VITE_GOOGLE_CLIENT_ID=...
```

## AI Model Requirements

- **Models:** `gemini-3-flash-preview` (High Complexity) and `gemini-3.1-flash-lite-preview` (Standard/OCR).
- **Selection Logic:**
  - `gemini-3-flash-preview`: Used for complex code generation (`mini-app`, `widget-builder`) and deep multimodal analysis (`guided-learning`).
  - `gemini-3.1-flash-lite-preview`: Used for standard JSON tasks (polls, quizzes, layouts) and high-speed multimodal tasks like `ocr`.
- **Status:** These models are **REQUIRED**. Older models (e.g., gemini-1.5-flash, gemini-2.0-flash) are deprecated and must not be used.

## Usage Limits

To manage costs and prevent abuse, AI features have the following usage limits:

- **Admins:** Unlimited usage.
- **Non-Admins:** 20 generations per day (resetting at midnight UTC).
- **Enforcement:** Limits are enforced server-side via Cloud Functions and tracked in the `ai_usage` Firestore collection.
- **Error Message:** Users who exceed the limit will see a clear error message: "Daily AI usage limit reached (20 generations). Please try again tomorrow."

## Recent Technical Improvements

- **Seating Chart Widget:** Fixed click propagation issues where furniture items would deselect immediately upon being clicked. Added unit tests for selection/deselection logic.
- **Weather Widget:** Enhanced admin configuration to support conditional ranges (Above, Below, and Between) with custom imagery.
- **Security:** Hardened Storage and Firestore rules; fixed "NaN" validation issues in admin configuration panels.
- **Performance:** Implemented lazy loading for all widget components to improve initial load performance.

## Project Memories

- **Strict Linting Policy:** Do not commit or push any code that introduces ESLint warnings or errors. Ensure that `pnpm run validate` succeeds before doing a final push or creating a PR.
- **Widget Scaling (Clock):** Perfectly satisfied with the Clock widget's scaling (fill-first logic with `min(75cqh, 25cqw)`). Do not change.
- **Widget Scaling (Weather):** Perfectly satisfied with the Weather widget's scaling (refined fill-first logic with cqh/cqw constraints and specific padding). Maintain as is.

## Widget Appearance Standard (Visual System)

All agents must follow the shared widget appearance model when building or updating configurable widgets:

- Use shared settings primitives in widget style tabs:
  - `components/common/TypographySettings.tsx`
  - `components/common/TextSizePresetSettings.tsx`
  - `components/common/SurfaceColorSettings.tsx`
- Prefer these config fields for visual controls:
  - `fontFamily`
  - `fontColor`
  - `textSizePreset` (`small` | `medium` | `large` | `x-large`)
  - `cardColor`
  - `cardOpacity`
- Keep universal transparency in the global settings shell; do not duplicate full-widget transparency controls inside widget-specific style tabs.
- Ensure front-face widgets actually consume settings values (no dead controls).
- Default widget roots should remain visually transparent; only add localized readability surfaces where content legibility requires it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OPS-PIvers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
