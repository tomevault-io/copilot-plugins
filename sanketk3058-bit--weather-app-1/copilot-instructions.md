## weather-app-1

> This file is the single source of truth for how automation agents (LLMs, scripts, CI bots) should work with this codebase.

# AGENTS.md - Weather App Project

## Build/Lint/Test Commands
# AGENTS.md – Weather App Project

This file is the single source of truth for how automation agents (LLMs, scripts, CI bots) should work with this codebase.

---

## 1. Build / Lint / Test

- `npm run dev` – Start development server (Next.js App Router)
- `npm run build` – Production build
- `npm run start` – Start production server
- `npm run lint` – ESLint (`next/core-web-vitals`)
- `npm run test` – Jest test suite (`__tests__/`)
- Single test: `npx jest --testNamePattern="test name"` or `npx jest path/to/test.tsx`

Always prefer `npm` (not `yarn`/`pnpm`) unless the repo changes.

---

## 2. Project Architecture (for Agents)

- **Framework**: Next.js 14 App Router, TypeScript, Tailwind CSS 4.
- **Client vs Server**:
	- Client components live under `src/app/page.tsx` and `src/components/*` and must start with `'use client'`.
	- API routes live under `src/app/api/*/route.ts` and must be the only place that talks to OpenWeatherMap.
- **Service Layer**: `src/lib/weather-api.ts` is the single integration point for OpenWeatherMap (current, forecast, air quality, UV, reverse geocoding, pollen generation).
- **Types**: `src/lib/types.ts` defines all public data contracts (weather, forecast, AQI, UV, pollen, location, etc.). New data must be typed here first.
- **State & Hooks**:
	- `useWeather` orchestrates client-side fetching and state (currently partially duplicated in `page.tsx`). Prefer consolidating future logic into the hook.
	- `useAnimatedNumber` handles numeric animations via Framer Motion.
- **Styling**:
	- Glass design via `glass-card` classes in `src/app/globals.css`.
	- Theming and dark/light behavior coordinated with `ThemeToggle` and body classes.

Before changing architecture, check `ARCHITECTURE_FLOW.md` and `TIMEZONE_COUNTRY_FIX_PLAN.md` for existing decisions.

---

## 3. Code Style & Patterns

### 3.1 Imports & Modules

- Use `@/*` alias for anything under `src/*` (configured in `tsconfig.json`).
- Import order (top → bottom): React/Next, third-party, local components/hooks/lib, then types.
- Prefer named exports for components and utilities; avoid default exports.

### 3.2 TypeScript & Types

- Strict TypeScript; **never** introduce `any`.
- Define all shared interfaces/types in `src/lib/types.ts`.
- Use descriptive type names and props interfaces suffixed with `Props` (e.g., `SearchBarProps`).
- Use string literal unions and `as const` for variants and discriminated unions (e.g., risk levels, card variants).
- When adding new API fields, update:
	- The relevant interfaces in `types.ts`.
	- The transformer functions in `weather-api.ts`.
	- Any API routes and UI components that consume those types.

### 3.3 Naming & Files

- Components/hooks: PascalCase (`GlassCard`, `HourlyForecast`, `useWeather`).
- Variables/functions: camelCase.
- Files: kebab-case for React components and utilities (`glass-card.tsx`, `input-sanitizer.ts`).

### 3.4 Components & Hooks

- All interactive React components must start with `'use client'`.
- Components must be small, focused, and reusable. Use composition rather than deep prop drilling.
- Use variants as string unions for styling (`'primary' | 'secondary' | 'tertiary'`, `animateHover` options).
- For new behavior that is reused, extract a hook in `src/hooks/` with clear responsibilities and types.

### 3.5 Error Handling & Resilience

- All optional/secondary API calls (air quality, UV, pollen) must use `fetchWithFallback()` and return safe defaults.
- Rate limiting and input sanitization live in `src/lib/rate-limit.ts` and `src/lib/input-sanitizer.ts`; do not bypass them from API routes.
- Never let a single failing optional endpoint break the full weather response – log and degrade gracefully.
- When logging, avoid dumping secrets or full URLs with API keys.

### 3.6 Styling & Animations

- Styling is Tailwind CSS + global CSS in `globals.css`.
- Glassmorphism is implemented using `glass-card` classes and variants; reuse them rather than rolling custom card styles.
- Hover/elevation effects must be CSS-based transforms (no Framer Motion on hover states).
- Framer Motion is only used where already established (e.g., `useAnimatedNumber`). Do not introduce new heavy animation patterns without need.

---

## 4. API / Data Flow Rules

- Components must **never** talk directly to OpenWeatherMap; they only call internal API routes.
- API routes under `src/app/api/*/route.ts`:
	- Validate and sanitize all input (lat/lon, query strings) via utilities in `src/lib`.
	- Delegate network calls to `weather-api.ts` only.
	- Use proper HTTP status codes and `NextResponse.json()`.
- `weather-api.ts` responsibilities:
	- Centralize axios configuration (base URL, 10s timeout, API key injection).
	- Provide typed helpers: current weather, forecast, air quality, UV, reverse geocode, timezone/country handling, and pollen generation.
	- Encapsulate transformation from raw OpenWeatherMap responses into internal `WeatherData` structures.
- Location enrichment:
	- Country must be populated from geocoding (see `TIMEZONE_COUNTRY_FIX_PLAN.md`).
	- Timezone must be derived from current weather response and exposed via types.

---

## 5. Testing & Quality

- Use Jest with `ts-jest` and `jest-environment-jsdom` (see `jest.config.cjs`).
- Tests live under `__tests__/` and should target critical logic (API transforms, hooks, and complex components).
- When adding non-trivial logic, add or extend tests rather than relying only on manual verification.
- Run `npm run lint` and `npm run test` before proposing large changes.

---

## 6. Best Practices for Future LLM Agents

These rules are for any future automation or LLM integrating with this repo.

0. **NEVER run git commands**: Do not execute any git commands (e.g., `git add`, `git commit`, `git push`, `git pull`, `git checkout`, `git branch`, `git stash`, `git diff`, `git status`, `git log`, etc.). The user manages version control manually. This rule has no exceptions.
1. **Read the instructions first**: Always load `.github/copilot-instructions.md`, `AGENTS.md`, and `ARCHITECTURE_FLOW.md` before editing code.
2. **Prefer minimal, targeted diffs**: Change only what is necessary; avoid broad refactors without explicit instruction.
3. **Respect layering**: Do not introduce cross-layer coupling (e.g., components calling external APIs or services bypassing types).
4. **Keep types in sync**: When API responses or shapes change, update `types.ts`, `weather-api.ts`, and any dependent code in a single coherent change.
5. **Preserve resilience**: Any new external data source must follow the same `fetchWithFallback` pattern and have safe default values.
6. **Avoid new dependencies**: Do not add npm packages unless absolutely required and consistent with project goals; prefer existing stack and utilities.
7. **No secrets in code**: Environment variables such as `OPENWEATHER_API_KEY` must stay in `.env.local` and never be hard-coded.
8. **Follow UI design language**: New UI elements should reuse `GlassCard`, Tailwind spacing/typography, and existing glassmorphism patterns.
9. **Do not duplicate logic**: If you find similar logic in multiple places (e.g., weather fetching in `page.tsx` and `useWeather`), prefer consolidating into a single well-tested abstraction.
10. **Document non-obvious decisions**: For architectural or behavioral changes, update `ARCHITECTURE_FLOW.md`, `TIMEZONE_COUNTRY_FIX_PLAN.md`, or create a new `*_PLAN.md` file.

---

## 7. When in Doubt

- Re-read `.github/copilot-instructions.md` for high-level project conventions.
- Check existing files for patterns and mimic their style.
- Prefer small, reversible changes with clear commit messages.
- Ask for clarification (or leave TODO comments) rather than guessing about business requirements.

This document should be kept up to date whenever the architecture, tooling, or coding standards meaningfully change.
- Styling: Backdrop-filter blur(40px), rgba backgrounds, subtle borders/shadows.
- Antipatterns: No direct API calls from components, no mixing CSS/Framer Motion.</content>
<parameter name="filePath">C:\Users\Admin\Desktop\Weather App Project\weather-app-project\AGENTS.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sanketk3058-bit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sanketk3058-bit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
