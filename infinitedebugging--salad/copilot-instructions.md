## salad

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.
Primary audience: autonomous coding agents.

## Project Snapshot

- App type: browser-only SPA (no backend).
- Stack: Vite + vanilla JavaScript (ES modules).
- Runtime target: modern browsers with `localStorage`, `Notification`, and DOM APIs.
- Data persistence: localStorage key `meal-chopping-app-v1`.
- Architecture style: centralized store + string-template UI rendering + event rebinding.

## Repository Rules Discovery

- Cursor rules: none found (`.cursorrules` and `.cursor/rules/` absent).
- Copilot instructions: none found (`.github/copilot-instructions.md` absent).

## Commands

Use npm for all package/script execution.

### Development

- Install deps: `npm install`
- Start dev server: `npm run dev`
- Build production bundle: `npm run build`
- Preview production build: `npm run preview`

### Linting

- No linter is currently configured.
- Do not invent lint commands in automation.

### Tests

- No test runner is currently configured.
- There is no `npm test` script at the moment.
- When adding tests, prefer Vitest for this Vite setup.

### Single Test Execution (Important)

Current state:
- Single-test execution is not available yet because no test runner is installed.

When Vitest is introduced, use one of these patterns:
- Single file: `npx vitest run src/path/to/file.test.js`
- Single test by name: `npx vitest run src/path/to/file.test.js -t "test name"`
- Watch one file: `npx vitest src/path/to/file.test.js`

## Agent Workflow Expectations

- Prefer small, focused patches instead of large rewrites.
- Keep modules responsibility-driven (state, timer, UI, storage, models).
- After meaningful code changes, run `npm run build` to verify no regressions.

## Bug Fix Workflow (Default)

Use failing-test-first when tests exist:
1. Reproduce with a failing test through a public interface.
2. Implement a single-pass fix (no parallel candidate patching by default).
3. Re-run the specific failing test.
4. Re-run relevant broader tests/build checks.

If tests do not exist:
- Add a focused test first when practical.
- If adding tests is out of scope, document manual reproduction and verification steps.

## Code Style and Conventions

### JavaScript Language and Modules

- Use ESM syntax (`import` / `export`) with explicit `.js` file extensions for local imports.
- Prefer named exports for reusable functions.
- Avoid default exports unless a module has a clear single primary export.
- Keep modules small and cohesive by domain responsibility.

### Imports

- Keep import lists stable and readable; avoid unused imports.
- Prefer relative imports within `src/` matching existing patterns.

### Formatting

- Use 2-space indentation.
- Use single quotes for strings.
- Do not use semicolons (match existing style).
- Keep trailing commas in multiline object/array/function literals.
- Keep lines readable; split long template strings and conditions sensibly.

### Naming

- `camelCase` for variables/functions.
- `UPPER_SNAKE_CASE` for exported constants (example: `PHASE_FOCUS`).
- UI renderer naming convention: `renderX` and `bindX` pairs.
- Use descriptive booleans with intent (`isExpanded`, `shouldUseLongBreak`).

### State Management

- Treat store state as immutable in action handlers; construct new objects/arrays.
- Keep persisted domain state separate from transient UI state.
- Persist only `meals`, `runs`, `phaseEntries`, and `settings`.
- Use centralized `commit`/`notify` flow rather than ad hoc state mutation.

### UI Patterns

- Render functions should return HTML strings only.
- Event listeners should be attached in paired `bind...` functions after render.
- Escape user-provided text before injecting into HTML templates.
- Keep DOM querying scoped and defensive (`if (element) { ... }`).

### Time/Timer Logic

- Keep phase transition rules in timer modules, not scattered across UI modules.
- Use helper utilities for formatting (`formatClock`, `formatDuration`, `formatDateTime`).

### Error Handling and Defensive Coding

- Validate and sanitize all localStorage reads.
- Always provide safe fallbacks for malformed persisted data.
- Guard browser-only APIs (`Notification`, `localStorage`, focus selection APIs).
- For unsupported capabilities, fail soft with user-visible notice rather than crashing.

### Data and Persistence

- Maintain localStorage schema compatibility (`version` field is present; keep it meaningful).
- When adding persisted fields, update sanitization and defaults together.
- Avoid destructive data migrations without explicit user request.

## Documentation Expectations

- Keep `README.md` aligned with actual commands and features.
- Record substantial decisions in `docs/decisions.md`.
- Record shipped progress in `docs/progress.md`.
- Add future work ideas to `docs/improvements.md`.

## What Not to Do

- Do not introduce frameworks unless explicitly requested.
- Do not add backend dependencies for core functionality.
- Do not bypass existing store architecture with scattered global state.
- Do not silently change persisted schema behavior.
- Do not claim lint/test results that were not actually executed.

## Pre-Completion Checklist for Agents

- Changes are scoped and modular.
- Imports are clean and local paths include `.js`.
- User text interpolation is escaped in rendered HTML.
- Persistence defaults/sanitizers updated for any new settings fields.
- `npm run build` succeeds.
- Docs updated when behavior or workflow changed.

---
> Source: [InfiniteDebugging/salad](https://github.com/InfiniteDebugging/salad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
