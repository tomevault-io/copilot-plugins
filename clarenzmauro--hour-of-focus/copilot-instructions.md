## hour-of-focus

> This document provides essential information for AI coding agents to operate efficiently and safely within the `hour-of-focus` codebase.

# Agent Instructions for hour-of-focus

This document provides essential information for AI coding agents to operate efficiently and safely within the `hour-of-focus` codebase.

## Project Overview

`hour-of-focus` is a minimalist React application designed to help users focus on a single high-impact project for 60 minutes. It features a three-phase workflow: goal setting, distraction preparation, and a 60-minute countdown timer.

## Technical Stack

- **Framework**: React 19
- **Build Tool**: Vite
- **Language**: TypeScript
- **Styling**: Vanilla CSS with system theme support
- **Package Manager**: bun

## Essential Commands

### Build and Development

- `bun run dev`: Starts the Vite development server.
- `bun run build`: Type-checks and builds the application for production.
- `bun run preview`: Previews the production build locally.

### Linting

- `bun run lint`: Runs ESLint across the codebase.
- This project uses the latest flat configuration (`eslint.config.js`).

### Testing

- `bun run test`: Runs unit tests using Vitest.
- `bun run test -- <filename>`: Runs a specific test file.
- Testing is configured with `vitest`, `@testing-library/react`, and `jsdom`.

## Code Style & Conventions

### TypeScript

- Use strict mode (configured in `tsconfig.app.json`).
- Prefer interfaces for object shapes, types for unions/aliases.
- Avoid `any`. Use `unknown` if the type is truly unknown.
- Define prop types for all components.

### React

- Use functional components and hooks exclusively.
- Prefer `useState` and `useEffect` for simple local state and side effects.
- Use `StrictMode` as seen in `main.tsx`.
- Follow React 19 patterns.

### Naming Conventions

- **Components**: PascalCase (e.g., `App.tsx`, `FocusTimer.tsx`).
- **Files**: kebab-case for all files (e.g., `app.tsx`, `focus-timer.tsx`).
- **Variables/Functions**: camelCase.
- **Constants**: UPPER_SNAKE_CASE.
- **CSS Classes**: kebab-case.

### Imports

- Use standard ES module imports.
- Import CSS files directly into the relevant component file.
- Group imports: React/library imports first, then local components/assets, then CSS.

### Styling

- Use `app.css` and `index.css`.
- Adhere to the minimalist aesthetic.
- Support system dark/light modes using `@media (prefers-color-scheme: dark)`.
- Use CSS variables if the project grows to maintain consistency.

### Error Handling

- Use standard `try/catch` for asynchronous operations or potentially throwing code.
- Ensure the UI provides feedback if something fails (though the current app is simple and mostly local).

## AI Agent Operating Procedures

1. **Read Before Writing**: Always read existing components and their corresponding CSS before making changes to ensure style consistency.
2. **Type Safety**: Maintain the strict TypeScript configuration. Never bypass type checks without a compelling reason and documentation.
3. **Minimalism**: The core value of this app is minimalism. Avoid adding unnecessary features, complex animations, or bloated dependencies.
4. **Tool Usage**:
   - Use `read` to understand the context of a file.
   - Use `edit` for surgical changes.
    - Use `bash` to run `bun run lint` or `bun run build` after changes to verify integrity.
5. **Testing**: Add or update unit tests for any new logic or components.
6. **Path Resolution**: Always use absolute paths when calling tools like `read`, `write`, or `edit`.
7. **No Local Execution**: Do not run the project using `bun run dev`. Instead, use `bun run build`, `bun run lint`, and any test commands to verify your changes.

## File Structure

```text
/
├── public/              # Static assets
├── src/
│   ├── assets/          # Images/icons
│   ├── app.tsx          # Main application logic and state
│   ├── app.test.tsx     # Main application tests
│   ├── setupTests.ts    # Test setup
│   ├── app.css          # Component-specific styles
│   ├── index.css        # Global styles
│   ├── main.tsx         # Entry point
│   └── vite-env.d.ts    # Vite type definitions
├── eslint.config.js     # Linting configuration
├── index.html           # HTML template
├── package.json         # Dependencies and scripts
├── tsconfig.json        # TypeScript configuration
└── vite.config.ts       # Vite configuration
```

## Future Recommendations for Agents

- If the logic in `app.tsx` becomes complex, refactor into smaller components (e.g., `GoalInput`, `Preparation`, `Timer`).
- Introduce a state management solution (like `Context API`) only if the application state grows beyond what `app.tsx` can cleanly handle.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/clarenzmauro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
