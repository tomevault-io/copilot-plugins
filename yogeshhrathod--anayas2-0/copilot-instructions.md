## anayas2-0

> - Desktop application built with Electron 28, Vite 5, React 18, and TypeScript for professional REST API testing and development.@package.json#1-61 @vite.config.ts#1-47 @README.md#1-128

# Cursor Rules

## Project Summary
- Desktop application built with Electron 28, Vite 5, React 18, and TypeScript for professional REST API testing and development.@package.json#1-61 @vite.config.ts#1-47 @README.md#1-128
- Electron main process initializes a JSON-backed datastore, registers IPC handlers, and loads the renderer. Node integration is disabled; preload exposes a typed bridge via `contextBridge`.@electron/main.ts#1-79 @electron/database/json-db.ts#27-65 @electron/ipc/index.ts#1-1 @electron/preload.ts#1-150
- Renderer is a React/Tailwind UI living under `src/` with Zustand for state management and Monaco-based editors for JSON templates.@src/main.tsx#1-10 @README.md#92-128

## Development Workflow
1. Install dependencies: `npm install`.
2. Run desktop dev mode: `npm run electron:dev` (starts Vite + Electron with hot reload).@package.json#8-17 @README.md#63-88
3. Type-check: `npm run type-check` before shipping changes.@package.json#8-18
4. Production build: `npm run build` (or platform-specific `build:mac|win|linux`).@package.json#8-18 @README.md#78-90

## Coding Guidelines
- Keep TypeScript strictness; prefer typed APIs in preload and IPC layers. Extend interfaces in `electron/preload.ts` when exposing new functionality.@electron/preload.ts#1-150
- All renderer-to-main communication must go through `window.electronAPI` helpers created in preload. Do **not** enable Node integration or access `ipcRenderer` directly from React components.@electron/preload.ts#61-150
- Register IPC handlers in `electron/ipc/handlers.ts` and export updates via `electron/ipc/index.ts`. Ensure handlers validate inputs and surface errors with descriptive logs.@electron/ipc/handlers.ts#22-31 @electron/ipc/index.ts#1-1 @electron/services/logger.ts#1-47
- Use the Winston logger for background services; avoid `console.log` in production paths. Configure module-specific loggers via `createLogger` helper.@electron/services/logger.ts#1-47
- Persist application state through the JSON DB helpers in `electron/database`. Add schema migrations or default seeding within `initDatabase` when required.@electron/database/json-db.ts#27-65
- Maintain strict separation between UI state (Zustand) and IPC contracts. Update shared types under `src/types/` when introducing new data structures.@README.md#92-128

## UI Conventions
- TailwindCSS governs styling with shadcn/ui primitives; respect existing utility classes and component patterns.
- Place shared UI pieces under `src/components/` and domain screens under `src/pages/`.
- Favor composition and hooks aligned with current store setup in `src/store/useStore.ts`.

## Feature Tracking & Spec-Driven Development

This project uses a **spec-driven development** workflow. All new features and bugs MUST follow this process:

### For Features:
1. **Create Feature**: Use `./scripts/create-feature.sh <feature-name>` to create a new feature in `specs/XXX-feature-name/`
2. **Write Spec**: Complete `spec.md` with user stories, acceptance criteria, and technical requirements
3. **Create Plan**: Complete `plan.md` with architecture decisions and implementation phases
4. **Break Down Tasks**: Complete `tasks.md` with specific, actionable tasks
5. **Implement**: Follow tasks.md, updating task status as you progress

### For Bugs:
1. **Create Bug**: Use `./scripts/create-bug.sh <bug-name>` to create a new bug in `specs/bug-XXX-bug-name/`
2. **Write Bug Report**: Complete `spec.md` with reproduction steps, expected/actual behavior, and environment details
3. **Create Fix Plan**: Complete `plan.md` with root cause analysis and fix strategy
4. **Break Down Fix Tasks**: Complete `tasks.md` with specific fix tasks and verification steps
5. **Implement Fix**: Follow tasks.md, updating bug status (reported → investigating → fixing → testing → resolved)

**ALWAYS** reference the feature/bug's spec.md, plan.md, and tasks.md when implementing. See `.cursor/rules/feature-tracking.mdc` for detailed workflow rules.

Link features to phases in `plan-timeline.md` using the `Phase` field in spec.md. Bug fixes can be linked to maintenance phases.

## Goal-Aware Development Workflow

**CRITICAL**: Before implementing any feature, ALWAYS follow this workflow. This ensures performance-first development and goal alignment.

### Step 1: SPEC (Do NOT write any code yet)

1. **Read context files FIRST**:
   - `/ai-context/project-goal.md` - Understand project goals and performance targets
   - `/ai-context/architecture.md` - Understand architecture patterns and performance strategies
   
2. **Create/update `spec.md`** in `specs/XXX-feature-name/`:
   - Include **Goal Alignment Summary** (how feature supports performance-first mission)
   - Include **Performance Impact Analysis** (memory, bundle size, load time estimates)
   - Include **Success Criteria** with performance metrics
   - Include **Constraints** (performance budgets, lazy loading requirements)
   - List any **unclear points** to confirm

3. **Wait for approval** before proceeding to Step 2.

### Step 2: PLAN (No code yet)

1. **Read context files**:
   - `/ai-context/example-quality.md` - Understand code quality standards and performance patterns
   - `/ai-context/common-utils.md` - Identify reusable utilities to avoid duplication

2. **Create/update `plan.md`** with:
   - **Exact files to modify/create** (with WHY for each file)
   - **Architecture alignment check** (follows architecture.md patterns)
   - **Reusable utilities identification** (uses common-utils.md)
   - **Lazy loading strategy** (how feature loads on-demand)
   - **Code splitting plan** (separate bundle if needed)
   - **Memory management plan** (cleanup strategy)
   - **Performance budget verification** (memory, bundle size, load time)
   - **Goal alignment check** (supports performance-first mission)

3. **Run goal alignment check**:
   - Does the plan support the long-term project goal? (Performance-first, low memory)
   - Are there more reusable or cleaner ways to achieve the same?
   - If something conflicts with the goal, warn the user.

4. **Wait for approval** before proceeding to Step 3.

### Step 3: IMPLEMENT (Write code)

1. **Follow plan exactly** - Do NOT modify files not listed in the plan.

2. **Reference context files**:
   - Use patterns from `example-quality.md`
   - Use utilities from `common-utils.md`
   - Follow architecture from `architecture.md`

3. **Implement performance tracking**:
   - Track memory usage before/after feature load
   - Track load time
   - Track bundle size impact
   - Log performance metrics

4. **After implementation, run self-review**:
   - Architecture violations?
   - Duplication?
   - Missing reusability?
   - Performance issues? (memory leaks, slow operations)
   - Missing lazy loading?
   - Missing cleanup?
   - Performance budget violations?

5. **Present improvements** but DO NOT apply them until user says "apply fixes".

### Permanent Rules

- **Never skip steps** - Always follow SPEC → PLAN → IMPLEMENT
- **Never guess file structure** - Ask when unsure
- **Follow project-goal.md above all** - Performance-first is the priority
- **Prioritize reusability, clarity, and maintainability**
- **Never sacrifice long-term architecture for a quick shortcut**
- **If a user request conflicts with the main goal, warn the user**
- **Always track performance** - Memory and load time (PRIMARY), bundle size (informational)

### Performance-First Checklist (MANDATORY for every feature)

**PRIMARY GOALS (Memory & Speed):**
- [ ] Feature can be lazy-loaded (not loaded upfront) - REQUIRED
- [ ] Estimated memory footprint: _____ MB (target: <50MB per feature) - PRIMARY
- [ ] Estimated load time: _____ ms (target: <200ms) - PRIMARY
- [ ] Memory cleanup plan defined - REQUIRED (prevents memory leaks)
- [ ] Performance tracking implemented (memory and load time) - REQUIRED
- [ ] No performance regression in existing features (memory/speed) - REQUIRED

**SUPPORTING (for lazy loading):**
- [ ] Code splitting plan defined (enables lazy loading, reduces initial memory)

**INFORMATIONAL (tracked but not blockers):**
- [ ] Estimated bundle size: _____ KB (tracked for awareness, not a blocker)

## Feature Implementation Checklist
1. **Check for existing feature/bug spec**: Look in `specs/` directory for related features or bugs. If none exists, create one using the workflow above.
2. **Read feature/bug documentation**: Review `spec.md`, `plan.md`, and `tasks.md` before starting implementation.
3. **Search existing codebase FIRST**: Before creating anything new, search for:
   - Existing feature specs in `specs/` directory for similar implementations
   - Existing components in `src/components/` that can be reused or extended
   - Existing hooks in `src/hooks/` with similar logic
   - Existing utilities in `src/lib/` (parsers, generators, formatters, etc.)
   - Existing types in `src/types/` that can be extended
   - Existing pages in `src/pages/` where the feature should be integrated
   - Existing buttons/actions that can be extended
   - Existing services in `electron/services/` that can be reused
   - Existing IPC handlers in `electron/ipc/handlers.ts` for similar functionality
4. **Identify integration points**: Determine where to add the feature (existing button, component, page, etc.) before creating new ones
5. **Reuse over create**: Always prefer extending existing components/utilities over creating duplicates
6. Define or update IPC handlers for new capabilities in `electron/ipc/handlers.ts` and expose them through preload APIs.@electron/ipc/handlers.ts#22-31 @electron/preload.ts#61-150
7. Wire renderer calls to `window.electronAPI` methods; keep contracts typed in both preload and React code.
8. Persist data through database helpers; add migrations/defaults in `initDatabase` as needed.@electron/database/json-db.ts#27-65
9. Log important lifecycle events and errors using Winston logger instances.@electron/services/logger.ts#1-47
10. Validate behavior in development (`npm run electron:dev`) and ensure type checks pass.
11. **AUTOMATICALLY update feature status**:
    - When starting: Update `spec.md`, `plan.md`, `tasks.md` headers: `planned` → `in-progress`
    - When all tasks completed: Update all headers: `in-progress` → `completed`
    - Update `plan-timeline.md`: Mark phase items as `[x]` completed, update percentages
    - Run `./scripts/update-feature-index.sh` to refresh the index
12. **Track task completion**: Update task status in `tasks.md` as you work (`pending` → `in-progress` → `completed`)

## Packaging & Distribution
- `electron-builder` outputs installers to `release/` with icons from `build/icons/`. Update branding before distribution.@package.json#63-95
- Keep `main` entry pointing to `dist-electron/main.js`; verify build emits both `dist/` (renderer) and `dist-electron/` (main/preload).@package.json#1-6 @vite.config.ts#1-47

## Operational Notes
- Logs live under the Electron user data path inside a rotating `logs/` directory; consult `error.log` for production issues.@electron/services/logger.ts#6-35 @README.md#211-223
- Database defaults (timezone, formats, polling) are seeded on first run—review these when cloning the project to new environments.@electron/database/json-db.ts#51-63 @README.md#196-210

## PR & Review Expectations
- Ensure new functionality includes IPC-to-renderer wiring, data persistence, and UI updates where applicable.
- Provide reproducible steps, reference relevant scripts, and attach log excerpts if bug fixes are involved.
- Run `npm run type-check` and applicable tests (add tests if introducing logic-heavy modules).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yogeshhrathod) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
