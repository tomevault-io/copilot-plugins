## ai-prototype

> This repository is a Vite + React + TypeScript app using Mantine.

# AGENTS.md

This repository is a Vite + React + TypeScript app using Mantine.
Follow these conventions when working here.

## Scope
- Applies to entire repo unless overridden by nested AGENTS.md.

## Quick Commands
- Install: `yarn install`
- Dev server: `yarn dev`
- Build: `yarn build`
- Preview build: `yarn preview`
- Lint: `yarn lint`

## Tests
- No test runner configured in `package.json`.
- If tests are added later, document per-package commands here.
- Single-test guidance: not applicable until a test runner exists.

## Formatting & Linting
- Use ESLint for validation: `yarn lint`.
- No explicit formatter configured; follow existing file formatting.
- Keep line length reasonable (~80–120 chars) unless existing file differs.

## Code Style
- Language: TypeScript + React (TSX) in `src/`.
- Use modern ES modules (`import`/`export`).
- Prefer named exports for helpers, default exports for components if consistent.
- Keep components small and focused; avoid large monolithic render blocks.

## Imports
- Order imports as:
  1. External packages
  2. Internal modules
  3. Local styles/assets
- Group with blank lines between sections.
- Use absolute/relative paths consistent with existing files.

## Naming Conventions
- Components: `PascalCase` (e.g., `MyWidget`).
- Hooks: `useSomething`.
- Files: match component names for TSX (`MyWidget.tsx`).
- Variables/functions: `camelCase`.
- Constants: `SCREAMING_SNAKE_CASE` for truly constant values.

## Types
- Prefer explicit types for public component props and exported functions.
- Use type aliases for props: `type MyProps = { ... }`.
- Avoid `any`; use `unknown` and narrow when needed.
- Prefer `React.FC` only if existing code uses it (currently not).

## React Patterns
- Use functional components.
- Keep hooks at top level; do not call conditionally.
- Prefer `useMemo`/`useCallback` only when necessary.

## Styling
- Global styles live in `src/index.css` and `src/App.css`.
- Prefer Mantine components and props for layout/styling first.
- Keep inline styles minimal; use them only for small layout tweaks.

## Error Handling
- Prefer explicit guards and early returns.
- Handle async failures with `try/catch` and surface user-safe messages.
- Avoid swallowing errors silently.

## Accessibility
- Use semantic HTML inside components.
- Ensure buttons/inputs have accessible labels.

## File Structure
- `src/App.tsx` is the app entry component.
- `src/main.tsx` boots React and mounts the app.
- Static assets belong in `src/assets` or `public`.

## Git Practices
- Keep commits focused and scoped.
- Do not commit build artifacts or secrets.

## Cursor/Copilot Rules
- No `.cursor/rules`, `.cursorrules`, or `.github/copilot-instructions.md` found.
- If added later, merge their rules into this document.

## Notes for Agents
- Ask before adding new dependencies.
- Prefer minimal changes and align with existing patterns.
- Avoid refactors unless requested or required to fix a bug.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/limichange) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
