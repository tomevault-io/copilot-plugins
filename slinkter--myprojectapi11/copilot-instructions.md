## myprojectapi11

> **Cat Gallery** is a high-fidelity React 19 application built with Vite 7, emphasizing **Clean Architecture** through **Feature-Sliced Design (FSD)**. It allows users to explore random cat images, manage favorites, and customize UI themes and typography.

# GEMINI.md - Instructional Context

## Project Overview
**Cat Gallery** is a high-fidelity React 19 application built with Vite 7, emphasizing **Clean Architecture** through **Feature-Sliced Design (FSD)**. It allows users to explore random cat images, manage favorites, and customize UI themes and typography.

### Core Tech Stack
- **Framework:** React 19 (Functional Components, Hooks)
- **Build Tool:** Vite 7 (ES Modules, path aliases)
- **State Management:** Redux Toolkit (Slices, Async Thunks, Facade pattern)
- **Styling:** Tailwind CSS v4 (Utility-first, semantic tokens)
- **Animations:** Framer Motion (LazyMotion, Reduced Motion support)
- **API Client:** Axios (Interceptors, normalized mappers)
- **Typing:** Strict JSDoc (Type-safety without TypeScript overhead)

---

## Architecture (Feature-Sliced Design)
The project follows a strict hierarchical structure:
- `src/app/`: Global configuration (Redux store).
- `src/features/`: Independent business modules (`cats`, `theme`, `font`).
    - Each feature contains: `api`, `adapters`, `services`, `redux`, `hooks` (facades), and `components`.
- `src/shared/`: Reusable, logic-agnostic code (`ui`, `hooks`, `components`, `utils`).
- `src/config/`: App-wide constants, environment validation, and motion settings.

**Constraint:** `shared` MUST NOT import from `features`. Features SHOULD NOT import from other features (use `app` or `shared` for common logic).

---

## Development Conventions

### 1. Tooling & Workflow
- **Package Manager:** `pnpm` exclusively. Do NOT use `npm` or `yarn`.
- **Linting:** 0 Warnings Policy. Run `pnpm run lint` before every commit.
- **Commit Style:** [Conventional Commits](https://www.conventionalcommits.org/) (e.g., `feat:`, `fix:`, `refactor:`).
- **Environment:** Requires `VITE_BASE_URL` and `VITE_API_KEY` (TheCatAPI).

### 2. Coding Standards
- **Typing:** Use strict **JSDoc** for all exports, hooks, and component props.
- **State Access:** Components MUST NOT use `useSelector` or `useDispatch` directly. Use **Facade Hooks** (e.g., `useCats`, `useTheme`).
- **Data Flow:** API response → `catMapper` (Adapter) → `Redux State` → `Facade Hook` → `Component`.
- **Styling:** Use Tailwind semantic tokens. Use `cn()` utility from `@shared/utils/cn` for class merging.
- **Animations:** Use `LazyMotion` with `domAnimation` from `@config/motionConfig.js`. Always support `useReducedMotion()`.

### 3. Path Aliases
- `@features/*` -> `src/features/*`
- `@shared/*` -> `src/shared/*`
- `@app/*` -> `src/app/*`
- `@config/*` -> `src/config/*`

---

## Key Commands

| Task | Command |
|------|---------|
| Install | `pnpm install` |
| Development | `pnpm run dev` |
| Build | `pnpm run build` |
| Lint | `pnpm run lint` |
| Preview Build | `pnpm run preview` |
| Deploy | `pnpm run deploy` (GitHub Pages) |

---

## Critical Files
- `AGENTS.md`: Detailed guide for AI agents (Build, Lint, Architecture).
- `src/features/cats/hooks/useCats.js`: Example of the Facade pattern.
- `src/features/cats/adapters/catMapper.js`: Data normalization logic.
- `src/config/env.js`: Environment variable validation.
- `src/shared/utils/cn.js`: Tailwind class merger utility.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Slinkter) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
