## worktime

> A React + TypeScript time tracking web app using Firebase, Tailwind CSS, and Vite.

# WorkTime - AI Assistant Guidelines

## Project Overview

A React + TypeScript time tracking web app using Firebase, Tailwind CSS, and Vite.

## Code Philosophy

### 1. Clean Code

- Write self-documenting code with meaningful names
- Keep functions small and focused (single responsibility)
- Avoid deep nesting; prefer early returns
- No magic numbers/strings; use named constants
- DRY (Don't Repeat Yourself) - extract common patterns

### 2. Architecture

- **Business logic in services** (`src/services/`), not in components
- **Feature folders** for services (e.g., `balance/`, `schedule/`, `scroll/`)
- **Components are presentational**: receive props, render UI, emit events
- **Custom hooks** for reusable stateful logic
- **Pure functions** are preferred - easier to test and reason about

### 3. Test-Driven Development (TDD)

When doing TDD:

1. Write tests first, before implementing the real code
2. Run tests to confirm they fail
3. Implement the minimum code to make tests pass
4. Refactor if needed
5. Run tests again to confirm they still pass

General testing guidelines:

- Keep tests focused and fast
- Pure functions in services are easier to test
- Tests live in `tests/` mirroring `src/` structure

### 4. React Best Practices (These Take Precedence)

- Functional components with hooks (no class components)
- Separate container logic from presentational components
- Keep components small and focused
- Lift state up only when necessary
- Use TypeScript strictly (avoid `any`)
- Props over context for explicit dependencies (when practical)

## Project Structure

```
src/
├── components/     # Reusable UI components
├── pages/          # Route-level components
├── services/       # Business logic (feature folders)
│   ├── auth/
│   ├── balance/
│   ├── scroll/
│   └── ...
└── types/          # TypeScript types

tests/
└── services/       # Mirrors src/services structure
```

## Commands

- **Dev server**: `npm run dev`
- **Run all tests**: `npm test`
- **Run specific tests**: `npm test -- --testPathPattern="<pattern>"`
- **Build**: `npm run build`
- **Lint**: `npm run lint`
- **Full check** (TypeScript + lint + tests): `npm run check` — **run this before committing**

## Tech Stack

- React 18 + TypeScript
- Vite (build tool)
- Tailwind CSS (styling)
- Firebase (auth + Firestore)
- Jest (testing)

## Git Workflow

- Show commit message drafts before committing
- Use semantic commit messages (feat:, fix:, refactor:, etc.)
- Don't push or create PRs without explicit approval

## Agent test login mode

To run the app in local dev with a hard-coded test user while still keeping real Firebase auth available:

1. Add these environment variables to `.env.local`:

```env
VITE_AGENT_TEST_AUTH_ENABLED=true
VITE_AGENT_TEST_EMAIL=agent-test@example.com
VITE_AGENT_TEST_PASSWORD=agent-test-password
```

2. Start dev server as usual:

```bash
npm run dev
```

3. On the login page, use the configured credentials (or click "Fill test credentials").

When `VITE_AGENT_TEST_AUTH_ENABLED` is not set to `true`, the app uses normal Firebase auth behavior.

> **Security warning**
>
> Agent test login mode is **only** for local development and must **never** be enabled in production.
> All `VITE_*` environment variables are embedded in the client bundle, so the test password is **not secret** and must not be treated as a secure credential.


## UI verification expectations

When making UI changes, always verify both:

- Desktop layout
- Mobile layout

This should be part of every agent-driven change validation.

Be explicit in validation notes:

- List every screen/route checked.
- If the change affects a shared layout/component, verify all impacted screens on desktop and mobile.
- For general UI-impacting changes, validate at least: `/login`, `/timesheet`, `/stats`, `/settings` on both desktop and mobile.
- If any screen is not verified, clearly state it as a gap.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robert-northmind) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
