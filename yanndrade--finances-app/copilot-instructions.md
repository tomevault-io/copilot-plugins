## finances-app

> Personal finance monorepo centered on a Windows desktop app with optional mobile access over LAN.

# AGENTS.md - Coding Agent Guidelines

## Project Overview

Personal finance monorepo centered on a Windows desktop app with optional mobile access over LAN.

Current packages:

- **`packages/frontend`** - React 19 + Vite SPA in TypeScript. Shared UI for desktop and LAN/mobile access.
- **`packages/backend`** - Python 3.13 FastAPI backend with event sourcing, SQLite projections, and LAN security middleware. Package manager: **uv**.
- **`packages/desktop`** - Tauri v2 shell branded as **MeuCofri**. Starts and monitors the backend, manages tray/autostart/updater behavior, and packages the Windows MSI.
- **`packages/shared`** - Shared contract scaffold. Today it mainly contains `version.txt` plus placeholder docs for future shared schemas/types/contracts.
- **`packages/video`** - Remotion package used for tutorial/demo video assets.

## Build / Dev / Test Commands

### Root Scripts

The repo currently ships **PowerShell** automation only.

```powershell
# Start frontend + Tauri desktop dev flow
./scripts/dev.ps1

# Build backend sidecar and copy it into the Tauri bundle
./scripts/build-backend-sidecar.ps1

# Build signed or unsigned Windows release artifacts
./scripts/build-release-windows.ps1
```

### Frontend (`packages/frontend`)

```powershell
# Dev server
npm run dev

# Type-check + production build
npm run build

# Run tests
npm run test

# Run a single test file
npx vitest run src/features/dashboard/dashboard-view.test.tsx

# Run tests matching a name
npx vitest run -t "loads the monthly summary"
```

Notes:

- `vite.config.ts` binds the dev server to `127.0.0.1:5173`.
- `vitest.config.ts` uses `jsdom`, globals, `src/test/setup.ts`, and a `30000` ms timeout.

### Backend (`packages/backend`)

All commands assume you are in `packages/backend`.

```powershell
# Install/update dependencies
uv sync

# Run server
uv run backend --host 127.0.0.1 --port 8000

# Run HTTPS locally with self-signed certs
uv run backend --host 127.0.0.1 --port 8000 --https

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_app.py

# Run a single test function
uv run pytest tests/test_app.py::test_health_endpoint_uses_application_layer

# Verbose test output
uv run pytest -v -s
```

### Desktop (`packages/desktop`)

```powershell
# Tauri desktop dev runtime
npm run dev

# Direct Tauri CLI passthrough
npm run tauri -- dev

# Production desktop build
npm run build
```

Notes:

- In development, the desktop runtime expects the frontend dev server on `http://127.0.0.1:5173`.
- The Rust host process starts the backend with `uv run backend --host 0.0.0.0 --port 8000`.

### Video (`packages/video`)

```powershell
# Remotion studio
npm run dev

# Bundle compositions
npm run bundle

# Render tutorial video
npm run render
```

## Frontend Code Style

### Formatting & Indentation

- 2-space indent for TS/TSX/JS/JSON/YAML. LF line endings.
- No repo-wide ESLint or Prettier config is enforced. Match surrounding code.
- Use `"double quotes"` for TypeScript strings.

### Imports

- CSS import first, then a blank line.
- React/library imports next.
- Internal imports after a blank line.
- Use `type` imports for type-only usage.
- Prefer the `@/` alias for internal modules when it improves clarity, but be aware the current codebase mixes alias and relative imports.

### Components

- Functional components only.
- Prefer `export function ComponentName(...)`.
- Define props as `type ComponentNameProps = { ... }` above the component.
- Shared primitives live in `src/components/ui/`.
- App-wide components live in `src/components/`.
- Feature-specific UI lives in `src/features/<feature>/`.
- Use `cn()` from `@/lib/utils` for conditional class composition.

### Naming Conventions

- Files: `kebab-case.tsx` / `kebab-case.ts`
- Components and types: `PascalCase`
- Functions and variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`

### Types

- TypeScript is strict. Avoid `any`.
- Prefer `type` aliases over `interface` in frontend code.
- API payloads generally use `snake_case` fields to mirror backend JSON.
- Frontend-only derived state can use `camelCase`.
- Monetary values are integer cents.

### State Management

- There is **no Zustand/Redux** in the current frontend.
- `src/App.tsx` owns most app-level UI state with `useState` and view composition.
- Data fetching is coordinated by `useAppDataOrchestrator`.
- Forms use React Hook Form + Zod.

### Error Handling

- API calls throw `ApiError(status, detail)`.
- Surface failures with toasts via the existing `showToast` / error-toast patterns.
- Use `getErrorMessage(error: unknown)` when extracting messages.
- Empty `catch {}` blocks are acceptable only for genuinely non-critical failures such as local preference persistence.

### Styling

- Tailwind CSS 3 with CSS custom properties drives the design system.
- `src/styles.css` is a major source of theme tokens and reusable UI classes.
- Prefer semantic tokens like `primary`, `success`, `warning`, `finance-*`, `lifecycle-*`, `surface-*`, and `chart-*`.
- Prefer design tokens in components. Raw hex values may still exist in theme-definition CSS where already established.
- Dark mode uses the `class` strategy on `<html>`.

### Testing

- Vitest + Testing Library.
- Globals are enabled.
- Setup file: `src/test/setup.ts`.
- The setup currently mocks `recharts` `ResponsiveContainer` and stubs `ResizeObserver`.
- Tests are co-located next to source files.

## Backend Code Style

### Architecture

- Primary layering is `domain -> application -> infrastructure -> interfaces`.
- HTTP composition happens in `interfaces/http/bootstrap.py` and `interfaces/http/app.py`.
- Routes are built by factory functions in `interfaces/http/routes/`.
- The backend also serves the built frontend SPA when `frontend/dist` is available.

### Event Sourcing & Persistence

- State-changing operations append immutable events to `events.db`.
- Read models are projected into `app.db`.
- The projector materializes multiple tables, not just a minimal cache.
- SQLite WAL mode is enabled for the event store.
- Alembic currently covers the event-store migration path; projection schema is largely bootstrapped in code.

### Python Conventions

- Python 3.13 with type hints throughout.
- Use `from __future__ import annotations` for forward refs where needed.
- Prefer dataclasses for immutable domain objects and `Protocol` for dependency boundaries when following existing patterns.
- Domain/service errors should be translated to `HTTPException` at the route layer.
- Pydantic models and `Field(...)` should define request/response validation.

### Security & LAN Behavior

- The backend enforces LAN security middleware for non-local requests.
- Remote LAN clients are validated by subnet/origin/device token rules.
- Desktop-only LAN configuration endpoints stay localhost-only.
- HTTPS mode can generate and use self-signed certificates.

### Testing

- Tests live in `packages/backend/tests/`.
- The suite is heavily HTTP/integration-oriented and uses FastAPI `TestClient`.
- `monkeypatch` is the standard stubbing tool.
- Test names follow `test_<description_in_snake_case>`.

## Desktop Notes

- The Tauri app is single-instance and uses tray, autostart, and updater plugins.
- Close requests hide to tray unless the app is actively quitting.
- Development backend traffic is health-checked on `127.0.0.1:8000`.
- Release builds store `app.db`, `events.db`, and TLS assets under the app local data directory, not the install directory.

## Shared Package Notes

- `packages/shared` is currently a contract/versioning scaffold, not a published TypeScript or Python package.
- `version.txt` is the current source of truth for shared contract compatibility.
- If you introduce concrete shared DTOs, schemas, or types there, update docs and versioning rules in the same change.

## Key Design Decisions

- Monetary values are stored and transmitted as **integer cents**.
- Timestamps are ISO 8601 strings in UTC.
- User-facing copy is **Brazilian Portuguese (pt-BR)**.
- Code, comments, and commits should stay in English.
- Views are lazy-loaded in the frontend with `React.lazy()` and `Suspense`.
- Desktop is the primary surface; mobile/LAN access is intentionally narrower and security-gated.

## Design Context

### Users

- Primary user is one person managing personal finances, mostly on Windows desktop and occasionally on mobile browser over LAN.
- Core jobs: log a transaction in under 10 seconds, review monthly cashflow, track card invoices, confirm recurring expenses, and close reimbursements.
- The interface must remain fast, scannable, and low-friction during frequent context switching.

### Brand Personality

- Trustworthy
- Efficient
- Calm

### Aesthetic Direction

- Desktop-first finance dashboard with dense but readable information.
- Mobile/LAN surface focuses on essential flows and larger touch targets.
- Use restrained gradients, semantic status colors, rounded cards, and subtle depth.
- Avoid playful banking tropes, excessive decoration, and effects that compete with numbers.

### Design Principles

1. Speed over ceremony.
2. Numbers first.
3. Semantic consistency.
4. Dense but readable.
5. Calm reliability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yanndrade) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
