## caller

> This document provides guidelines and context for developing the Gemini CLI project. It is used by both human developers and the Gemini AI assistant to ensure consistency and adherence to project standards. As an AI assistant, I use this file as my primary source of truth for project-specific conventions, commands, and architecture.

# Gemini CLI Development Guide

This document provides guidelines and context for developing the Gemini CLI project. It is used by both human developers and the Gemini AI assistant to ensure consistency and adherence to project standards. As an AI assistant, I use this file as my primary source of truth for project-specific conventions, commands, and architecture.

## Project Overview

The Gemini CLI is an AI-powered phone calling assistant that integrates with services like LiveKit and Twilio. It features a TypeScript backend, a React frontend, and a worker process for handling tasks.

## Development

This section outlines the key commands for building, running, and testing the application. All commands are run using `npm run <command-name>`.

### Initial Setup

To get started, install all dependencies and run the initial database migration:

```bash
npm run setup
```

### Running the Development Environment

The main command to run all parts of the application (backend server, frontend, worker, and supporting services) in development mode is:

```bash
npm run dev
```

This command uses `concurrently` to run the following scripts:
- `dev:server`: Starts the backend server with hot-reloading.
- `dev:worker`: Starts the worker process with hot-reloading.
- `dev:frontend`: Starts the frontend development server.
- `tunnel:http` & `tunnel:sip`: Exposes local services to the internet using ngrok.
- `docker:dev`: Starts required infrastructure (LiveKit, Redis, etc.) using Docker.

You can also run these services individually if needed.

### Building for Production

To build the application for production, which cleans the `dist` directory and compiles the TypeScript code:

```bash
npm run build
```

### Starting in Production

To run the pre-built application:

```bash
# Starts the main server
npm run start

# Starts the worker
npm run start:worker
```

### Testing

The project uses Vitest for testing.

- `npm test`: Run all tests once.
- `npm test:watch`: Run tests in watch mode.
- `npm test:coverage`: Run tests and generate a coverage report.
- `npm test:e2e`: Run end-to-end tests for the frontend.

### Validation and Linting

To ensure code quality, use the following commands:

- `npm run validate`: The main pre-flight check. It runs `format:check`, `lint`, `typecheck`, and `test`. Run this before submitting changes.
- `npm run lint`: Check for linting errors.
- `npm run lint:fix`: Automatically fix linting and formatting errors.
- `npm run format`: Format the code using Prettier.
- `npm run typecheck`: Check for TypeScript errors.

### Other Useful Commands

- `npm run clean`: Removes build artifacts and caches.
- `npm run kill`: Stops all running processes started by `npm run dev`.
- `npm run db:migrate`: Manages database schema migrations.
- `npm run docker:up` / `npm run docker:down`: Manages Docker containers.

### Git

The main branch for this project is `main`.

## Writing Tests

This project uses **Vitest** as its primary testing framework. When writing
tests, aim to follow existing patterns. Key conventions include:

### Test Structure and Framework

- **Framework**: All tests are written using Vitest (`describe`, `it`, `expect`,
  `vi`).
- **File Location**: Test files (`*.test.ts` for logic, `*.test.tsx` for React
  components) are co-located with the source files they test.
- **Configuration**: Test environments are defined in `vitest.config.ts` files.
- **Setup/Teardown**: Use `beforeEach` and `afterEach`. Commonly,
  `vi.resetAllMocks()` is called in `beforeEach` and `vi.restoreAllMocks()` in
  `afterEach`.

### Mocking (`vi` from Vitest)

- **ES Modules**: Mock with
  `vi.mock('module-name', async (importOriginal) => { ... })`. Use
  `importOriginal` for selective mocking.
  - _Example_:
    `vi.mock('os', async (importOriginal) => { const actual = await importOriginal(); return { ...actual, homedir: vi.fn() }; });`
- **Mocking Order**: For critical dependencies (e.g., `os`, `fs`) that affect
  module-level constants, place `vi.mock` at the _very top_ of the test file,
  before other imports.
- **Hoisting**: Use `const myMock = vi.hoisted(() => vi.fn());` if a mock
  function needs to be defined before its use in a `vi.mock` factory.
- **Mock Functions**: Create with `vi.fn()`. Define behavior with
  `mockImplementation()`, `mockResolvedValue()`, or `mockRejectedValue()`.
- **Spying**: Use `vi.spyOn(object, 'methodName')`. Restore spies with
  `mockRestore()` in `afterEach`.

### Commonly Mocked Modules

- **Node.js built-ins**: `fs`, `fs/promises`, `os` (especially `os.homedir()`),
  `path`, `child_process` (`execSync`, `spawn`).
- **External SDKs**: `@google/genai`, `@modelcontextprotocol/sdk`.
- **Internal Project Modules**: Dependencies from other project packages are
  often mocked.

### Asynchronous Testing

- Use `async/await`.
- For timers, use `vi.useFakeTimers()`, `vi.advanceTimersByTimeAsync()`,
  `vi.runAllTimersAsync()`.
- Test promise rejections with `await expect(promise).rejects.toThrow(...)`.

### General Guidance

- When adding tests, first examine existing tests to understand and conform to
  established conventions.
- Pay close attention to the mocks at the top of existing test files; they
  reveal critical dependencies and how they are managed in a test environment.

## Style and Conventions

This section details the coding style and conventions for the project. The goal is to maintain a readable, consistent, and maintainable codebase.

### JavaScript/TypeScript

- **Prefer Plain Objects over Classes**: Use plain JavaScript objects with TypeScript `interface` or `type` declarations instead of ES6 classes. This improves React integration, reduces boilerplate, simplifies data flow, and encourages immutability.
- **Use ES Modules for Encapsulation**: Rely on `import`/`export` to define clear public APIs for modules. Unexported members are considered private. This enhances testability by encouraging tests to focus on public APIs and reduces coupling between modules.
- **Prioritize Type Safety**:
  - **Avoid `any`**: Do not use the `any` type, as it disables type checking.
  - **Prefer `unknown`**: When a type is truly unknown, use `unknown` and perform type-narrowing (e.g., `typeof`, `instanceof`) before operating on the value.
  - **Use Type Assertions (`as`) Sparingly**: Only use type assertions when you have more information than the compiler, and be aware that they bypass type safety. Needing assertions in tests can be a "code smell" indicating a need for refactoring.
- **Embrace Functional Array Methods**: Use immutable array methods like `.map()`, `.filter()`, and `.reduce()` over imperative loops (e.g., `for`). This improves readability and predictability by promoting a functional, side-effect-free style.

### React

#### Assistant's Role in React Development

When assisting with React code, the AI assistant will act as a specialist in writing efficient and optimizable React code. It will identify patterns that enable React Compiler to automatically apply optimizations, reducing unnecessary re-renders and improving application performance.

#### Core Principles

- **Use Functional Components and Hooks**: All new components should be function components using Hooks. Avoid class components and legacy lifecycle methods.
- **Keep Components Pure**: Render logic should be a pure function of props and state. Defer side effects (e.g., data fetching, subscriptions) to `useEffect` or event handlers.
- **Immutable State Updates**: Never mutate state directly. Always use the state setter function from `useState` or `useReducer` with new objects or arrays.
- **Follow the Rules of Hooks**: Call Hooks only at the top level of function components or custom Hooks. Do not call them inside loops, conditions, or nested functions.
- **Explicit `useEffect` Dependencies**: Include all reactive values used inside `useEffect` in its dependency array. Avoid using it for "do this when this changes" logic; prefer event handlers for user actions.
- **Minimize `useEffect` Usage**: `useEffect` is for synchronizing with external systems. Avoid it for simple state transformations that can be derived during rendering. Do not call state setters inside a `useEffect` as it can degrade performance.
- **Prefer Composition**: Break down complex UIs into small, reusable components. Extract repetitive logic into custom Hooks.
- **Optimize for Concurrency**: Write components that are resilient to being rendered multiple times. Use functional state updates (e.g., `setCount(c => c + 1)`) to prevent race conditions.
- **Let the Compiler Optimize**: Avoid premature optimization with `useMemo`, `useCallback`, and `React.memo`. Write simple, clear components and rely on the React Compiler for memoization.
- **Good UX for Data Loading**: Use lightweight placeholders (like skeleton screens) instead of spinners. Handle errors gracefully. Use Suspense to declaratively manage loading states.


## Documentation guidelines

When working in the `/docs` directory, follow the guidelines in this section:

- **Role:** The AI assistant will act as an expert technical writer for contributors
  to Gemini CLI, producing professional, accurate, and consistent documentation.
- **Technical Accuracy:** Do not invent facts, commands, code, API names, or
  output. All technical information specific to Gemini CLI must be based on code
  found within this directory and its subdirectories.
- **Style Authority:** Your source for writing guidance and style is the
  "Documentation contribution process" section in the root directory's
  `CONTRIBUTING.md` file, as well as any guidelines provided this section.
- **Information Architecture Consideration:** Before proposing documentation
  changes, consider the information architecture. If a change adds significant
  new content to existing documents, evaluate if creating a new, more focused
  page or changes to `sidebar.json` would provide a better user experience.
- **Proactive User Consideration:** The user experience should be a primary
  concern when making changes to documentation. Aim to fill gaps in existing
  knowledge whenever possible while keeping documentation concise and easy for
  users to understand. If changes might hinder user understanding or
  accessibility, proactively raise these concerns and propose alternatives.

## Comments policy

Only write high-value comments if at all. Avoid talking to the user through
comments.

## General requirements

- If there is something you do not understand or is ambiguous, seek confirmation
  or clarification from the user before making changes based on assumptions.
- Use hyphens instead of underscores in flag names (e.g. `my-flag` instead of
  `my_flag`).
- Always refer to Gemini CLI as `Gemini CLI`, never `the Gemini CLI`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/crsmithdev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/crsmithdev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
