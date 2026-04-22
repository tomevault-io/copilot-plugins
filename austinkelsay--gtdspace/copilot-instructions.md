## gtdspace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

```bash
# Development
npm run tauri:dev      # Primary dev server with hot reload
npm run dev            # Vite frontend only
npm run preview        # Preview production build

# Code Quality - ALWAYS RUN BEFORE COMMITTING
npm run type-check     # TypeScript validation
npm run lint           # ESLint check (CI fails on warnings)
npm run lint:fix       # Auto-fix ESLint issues
cd src-tauri && cargo check && cargo clippy  # Rust checks
cd src-tauri && cargo fmt  # Auto-fix Rust formatting

# Building
npm run tauri:build    # Creates platform installers
npm run build          # Frontend build only

# Testing
npm test               # Run Vitest in watch mode
npm run test:run       # Run tests once

# Release Management
npm run release:patch  # 0.1.0 → 0.1.1 with git tag
npm run release:minor  # 0.1.0 → 0.2.0 with git tag
npm run release:major  # 0.1.0 → 1.0.0 with git tag
```

## Pre-Commit Gate

Do not commit or ask for review until the relevant local checks pass.
Do not push if you made additional edits after the last successful run; rerun the relevant checks first.

- Always run after code changes:

  ```bash
  npm run type-check
  npm run lint
  cd src-tauri && cargo fmt --check && cargo clippy -- -D warnings
  ```

- If you touched `src/App.tsx`, file watcher handling, reload toasts, or integration flows, also run:

  ```bash
  npx vitest run tests/app.integration.spec.tsx
  ```

- If you touched MCP server, Rust backend settings, or path-resolution logic, also run:

  ```bash
  cargo test --manifest-path src-tauri/mcp-server/Cargo.toml --test gtdspace_mcp_stdio -- --nocapture
  cargo test --manifest-path src-tauri/Cargo.toml parse_user_settings_value_accepts_partial_saved_settings -- --nocapture
  ```

- If you touched frontend settings validation or MCP settings UI, also run:

  ```bash
  npx vitest run tests/settings-validation.spec.ts
  ```

- If you touched dashboard actions, overview, or projects, also run:

  ```bash
  npx vitest run tests/dashboard-actions.component.spec.tsx tests/dashboard-projects.component.spec.tsx tests/date-formatting.spec.ts
  ```

- If `cargo fmt` changes files, rerun the affected checks before committing.
- Treat CI-only failures caused by missing local verification as avoidable regressions.

## Setup Requirements

- **Node.js**: v20+ required
- **Rust**: Latest stable via [rustup](https://rustup.rs/)
- **First run**: `npm install` then `npm run tauri:dev`
- **Google Calendar**: Configure OAuth in Settings UI (or `.env` with `GOOGLE_CALENDAR_CLIENT_ID` and `GOOGLE_CALENDAR_CLIENT_SECRET`)

### Claude AI Settings Configuration

If you're using Claude Code (claude.ai/code) with this repository, you'll need to configure local settings:

1. **Copy the example file**:
   ```bash
   cp .claude/settings.local.json.example .claude/settings.local.json
   ```

2. **Edit `.claude/settings.local.json`** and replace the placeholder paths:
   - Replace `/path/to/your/workspace/**` with your actual workspace absolute path (e.g., `/Users/yourname/Desktop/code/gtdspace/**`)
   - If you need access to other directories, add additional `Read()` entries with specific paths

3. **Permissions are scoped to essential commands only**:
   - Development: `npm run tauri:dev`, `npm run dev`, `npm run build`
   - Code quality: `npm run type-check`, `npm run lint`, `cargo check`, `cargo clippy`, `cargo fmt`
   - Testing: `npm test`, `npm run test:run`
   - Git operations: `git add`, `git commit` (for PR work)
   - Utilities: `find`, `grep`, `cat`, `ls`, `rg`, `jq`, `diff`, `sed`, `rm`, `mkdir`

4. **Important**: 
   - `.claude/settings.local.json` is gitignored and contains machine-specific paths
   - Never commit your local settings file
   - The example file (`.claude/settings.local.json.example`) is sanitized and safe to commit
   - If you need additional permissions, add them to your local file only, not the example

## Architecture Overview

**Stack**: React 18 + TypeScript + Vite + Tauri 2.x (Rust backend) + BlockNote editor v0.37 (pinned)
**State Management**: Custom React hooks only - no Redux/Zustand
**Entry Points**: `src/App.tsx` (frontend), `src-tauri/src/lib.rs` (backend commands)

### Project Structure

```
src/
├── components/     # React UI (editor/, sidebar/, calendar/, dashboard/)
├── hooks/          # Business logic and state management
├── utils/          # Helpers (metadata-extractor.ts, date-formatting.ts, etc.)
├── lib/            # Shared utilities and configurations
src-tauri/
├── src/commands/   # Rust IPC commands
├── src/lib.rs      # Command registration (NOT main.rs)
scripts/            # Build automation (bump-version.js, safe-release.js)
```

### GTD Workspace Structure

Auto-created at `~/GTD Space` on first run:
- **Projects/** - Folders containing README.md + action files (.md)
- **Habits/** - Recurring tasks with frequency-based auto-reset
- **Purpose & Principles/**, **Vision/**, **Goals/**, **Areas of Focus/** - GTD horizons
- **Cabinet/**, **Someday Maybe/** - Reference and future ideas

## Core React Hooks

- **`useGTDSpace`**: Project/action CRUD operations, workspace initialization
- **`useTabManager`**: Multi-tab editor state (manual save with Cmd/Ctrl+S)
- **`useFileManager`**: Tauri file operations wrapper
- **`useCalendarData`**: Aggregates all dated items (actions, habits, Google Calendar)
- **`useHabitTracking`**: Auto-reset based on frequency with history logging
- **`useErrorHandler`**: Wraps all Tauri invokes with error handling
- **`useActionsData`**: Aggregates and filters actions across projects
- **`useProjectsData`**: Project metadata and hierarchy management
- **`useHorizonsRelationships`**: Manages references between GTD horizons

## Important Patterns

### Tauri Command Pattern

```typescript
import { invoke } from "@tauri-apps/api/core";
import { withErrorHandling } from "@/hooks/useErrorHandler";

// Frontend camelCase → Backend snake_case (auto-converted)
const result = await withErrorHandling(
  async () => await invoke<ReturnType>("command_name", { param }),
  "Error message"
);
```

### Content Event Bus

Window-level events for cross-component updates:
```typescript
// Dispatch
window.dispatchEvent(new CustomEvent('content-updated', { detail: { path } }));

// Listen
window.addEventListener('content-updated', handler);
```

Events: `content-updated`, `gtd-project-created`, `file-renamed`, `habit-status-updated`, `habit-content-changed`, `theme-changed`, `insert-actions-list`

### Window-Level Callbacks

Global callbacks registered on window object for cross-component communication:

```typescript
// Notifies when a tab file is saved (used to reload projects)
window.onTabFileSaved: (
  filePath: string,
  fileName: string,
  content: string,
  metadata: Record<string, unknown>
) => void

// Updates backlink references in open tabs without saving
window.applyBacklinkChange: (targetPath: string, mutator: (content: string) => string) => { handled: boolean; wasDirty: boolean }
```

These callbacks are set up in App.tsx and consumed by hooks/components throughout the app.

### Custom Markdown Fields

```markdown
[!singleselect:status:in-progress]     # Status dropdown
[!singleselect:effort:medium]          # Effort selector
[!datetime:due_date:2025-01-20]        # Date/time picker
[!checkbox:habit-status:false]         # Habit tracking
[!multiselect:contexts:home,work]      # Multiple tags
[!references:file1.md,file2.md]        # File links
[!actions-list]                        # Dynamic action list
```

## Key Features & Components

### Dashboard System

Five-tab layout in `src/components/dashboard/`:
- **Overview**: System statistics, trends, overdue alerts
- **Actions**: All actions with filtering (status/effort/dates/contexts)
- **Projects**: Portfolio view with progress tracking
- **Habits**: Tracking with streaks and auto-reset
- **Horizons**: GTD hierarchy visualization

### Specialized Page Components

The app routes to specialized editors based on file path and content detection:

- **`ActionPage`**: Actions under `Projects/*/` (not README.md), detected by status/effort fields
- **`HabitPage`**: Files in `Habits/` with habit-status/habit-frequency fields
- **`AreaPage`**: Files in `Areas of Focus/` with area-status/area-review-cadence fields
- **`GoalPage`**: Files in `Goals/` with goal-status/goal-target-date fields
- **`VisionPage`**: Files in `Vision/` with vision-horizon fields
- **`PurposePage`**: Files in `Purpose & Principles/` with reference fields
- **`EnhancedTextEditor`**: Fallback for all other markdown files (README.md, Cabinet, Someday Maybe)

**Routing Logic** (in `App.tsx:900-1066`):
1. Path heuristic: Check if file path matches expected directory structure
2. Content heuristic: Check if content contains characteristic metadata fields
3. Combine both: File qualifies if path OR content matches (allows flexibility)
4. Route to most specific component, fallback to EnhancedTextEditor

### Actions List

- Projects show expandable action lists in sidebar
- Real-time status updates (in-progress/waiting/completed)
- Insert `[!actions-list]` with Ctrl/Cmd+Alt+L

## Adding New Features

**New GTD Field**:
1. BlockNote component in `src/components/editor/blocks/`
2. Insertion hook in `src/hooks/use[FieldName]Insertion.ts`
3. Update `preprocessMarkdownForBlockNote()` in editor preprocessing
4. Add pattern to `DEFAULT_EXTRACTORS` in `src/utils/metadata-extractor.ts`
5. Register keyboard shortcut in `useKeyboardShortcuts`
6. Update type definitions in `src/types/` if needed

**New Tauri Command**:
1. Implement in `src-tauri/src/commands/` (create new module or add to existing)
2. Register in `src-tauri/src/lib.rs` invoke_handler (NOT main.rs)
3. Wrap with `withErrorHandling()` in frontend calls
4. Use snake_case in Rust, camelCase in TypeScript (auto-converted)

**New Specialized Page Component**:
1. Create component in `src/components/gtd/[Name]Page.tsx`
2. Add detection logic to `App.tsx` routing (lines 900-1066)
3. Define path heuristic (e.g., files in `Horizon Name/` directory)
4. Define content heuristic (e.g., specific metadata field patterns)
5. Add to routing switch before fallback to EnhancedTextEditor
6. Implement template/seeding in backend if needed

## Technical Constraints

- **TypeScript**: Strict mode disabled, path aliases configured (`@/*` → `./src/*`)
- **ESLint**: v9 flat config, **zero warnings allowed** (CI enforced), unused vars must start with `_`
- **Rust**: Must pass `cargo clippy -D warnings` and `cargo fmt --check`
- **BlockNote**: v0.37 pinned (DO NOT upgrade without thorough testing)
- **Node**: v20+ required, npm v9 package manager
- **Vitest**: Tests in `tests/` directory, run with `npm test`

## Key Utilities

**Date Handling** (`src/utils/date-formatting.ts`): `formatRelativeDate()`, `formatCompactDate()`, `formatRelativeTime()`
**Metadata** (`src/utils/metadata-extractor.ts`): `extractMetadata()`, `extractProjectStatus()`, `extractActionStatus()`
**Notifications**: `const { toast } = useToast()` for user feedback
**Error Handling**: Always wrap Tauri invokes with `withErrorHandling()` from `useErrorHandler`

## Critical Event Flows

- **Save**: Cmd/Ctrl+S → `saveTab()` → Tauri `write_file` → `window.onTabFileSaved()` → reload projects if needed → toast notification
- **File Watch**: External changes detected with 500ms debounce → UI auto-refresh → close/reload tabs
- **Content Bus**: Window-level events (`content-updated`, `gtd-project-created`, `file-renamed`, `habit-status-updated`)
- **Tab System**: Multi-tab with manual save, max 10 tabs, drag-and-drop reordering
- **Habit Reset**: 1-minute interval checks for expired habits → auto-reset based on frequency → refresh UI
- **File Routing**: File opened → detect type (path + content) → route to specialized component → render appropriate editor

## Performance Notes

- **Debouncing**: Auto-save (2s), metadata (500ms), file watcher (500ms)
- **Limits**: Max 10MB files, max 10 open tabs
- **Parallel Operations**: Calendar data loads, multiple file reads

## CI/CD & Release Process

**GitHub Actions**: `ci.yml` (linting/type checks), `build.yml` (platform builds), `release.yml` (creates releases)

**Release Commands**:
- `npm run release:patch/minor/major` - Full release with git tag
- `npm run version:patch/minor/major` - Version bump only
- Script updates package.json, Cargo.toml, and tauri.conf.json in sync

## Build Notes

- **Icons**: Auto-generated from `app-icon.png` before build
- **Platforms**: macOS (.dmg), Windows (.msi), Linux (.AppImage/.deb)
- **Versions**: Synchronized across package.json, Cargo.toml, tauri.conf.json

## Troubleshooting

- **Module errors**: `npm install` and restart
- **Rust errors**: `rustup update`
- **Calendar sync**: Check OAuth config in Settings or `.env` file
- **File watch issues**: Check 500ms debounce in console logs

## Commit Guidelines

From AGENTS.md (full repo guidelines):
- Use imperative mood with concise scope (e.g., `feat: add calendar week view`)
- Reference issues when applicable (e.g., `#123`)
- Ensure `npm run type-check` and `npm run lint` pass before committing
- Include screenshots for UI changes in PRs
- Rust changes must pass `cd src-tauri && cargo clippy -D warnings && cargo fmt --check`

## Path Normalization Utilities

When comparing file paths across platforms (Windows/Unix):

```typescript
// In App.tsx, used throughout the codebase
function norm(p?: string | null): string | null | undefined {
  return p?.toLowerCase().replace(/\\/g, '/');
}

function isUnder(p?: string | null, dir?: string | null): boolean {
  // Checks if path p is under directory dir (case-insensitive, normalized)
}
```

Always normalize paths before comparison to avoid cross-platform bugs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AustinKelsay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
