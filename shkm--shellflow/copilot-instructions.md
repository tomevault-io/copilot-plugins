## shellflow

> A Tauri desktop app for orchestrating git worktrees with integrated terminal support.

# Shellflow

A Tauri desktop app for orchestrating git worktrees with integrated terminal support.

## Tech Stack

- **Frontend**: React 19, TypeScript, Tailwind CSS, xterm.js
- **Backend**: Rust, Tauri 2.x
- **Build**: Vite, Cargo

## Project Structure

```
src/                    # React frontend
  components/           # UI components
  hooks/                # React hooks (useWorktrees, usePty, useGitStatus)
  lib/                  # Tauri invoke wrappers
  types/                # TypeScript types
src-tauri/              # Rust backend
  src/
    lib.rs              # Tauri commands
    worktree.rs         # Git worktree operations
    pty.rs              # Terminal/PTY management
    watcher.rs          # File system watcher
    config.rs           # User configuration
    state.rs            # App state types
```

## Development

```bash
npm install
npm run tauri dev
```

**Important**: Do not run the app unless asked to verify logs or test specific behavior. The user will typically run and test the app themselves.

## Actions

When adding new functionality, implement it as an **action** so it appears in the command palette. This ensures all features are discoverable and accessible via keyboard. Only skip this if there's a good reason (e.g., the feature is purely internal or doesn't make sense as a user-invokable action).

### Adding a New Action

Actions use namespaced format (e.g., `diff::open`, `worktree::new`). To add a new action, update these files:

1. **`src/lib/actions.ts`** - Action registry (3 places):
   - Add to `ActionId` type union
   - Add availability predicate in `AVAILABILITY` record
   - Add metadata in `ACTION_METADATA` (label, category, showInPalette)

2. **`src/lib/actionHandlers.ts`** - Handler wiring (2 places):
   - Add callback to `ActionHandlerCallbacks` interface
   - Add mapping in `createActionHandlers()` function

3. **`src/lib/defaultMappings.jsonc`** - Keyboard shortcut:
   - Add binding in appropriate context section

4. **`src/App.tsx`** - Implementation (3 places):
   - Create handler function (e.g., `handleOpenDiff`)
   - Add to `actionHandlers` useMemo
   - Add to `contextActionHandlers` useMemo (with `createActionHandlers()` call)

### Context Flags

Available context flags for keybindings (`src/lib/contexts.ts`):
- `scratchFocused`, `worktreeFocused`, `projectFocused`
- `drawerFocused`, `mainFocused`
- `drawerOpen`, `rightPanelOpen`
- `pickerOpen`, `commandPaletteOpen`, `modalOpen`
- `diffViewOpen`, `canGoBack`, `canGoForward`

### ActionContext

Available context for availability predicates:
- `activeProjectId`, `activeWorktreeId`, `activeScratchId`, `activeEntityId`
- `isDrawerOpen`, `isDrawerFocused`, `activeDrawerTabId`
- `openEntityCount`, `canGoBack`, `canGoForward`
- `isViewingDiff`, `changedFilesCount`
- `activeSelectedTask`, `taskCount`

## Testing

**Always write tests for new functionality and bug fixes.** After making changes, explicitly state whether tests were added and run them to verify they pass.

### Running Tests

```bash
# TypeScript/React tests (Vitest)
npm test

# Rust tests
cd src-tauri && cargo test
```

### Test Structure

- **Frontend tests**: `src/**/*.test.ts(x)` - Uses Vitest with mocked Tauri APIs
- **Backend tests**: `src-tauri/src/*.rs` - Uses `#[cfg(test)]` modules with `#[test]` functions

### Test Guidelines

1. Add tests for new features covering the happy path and edge cases
2. Add regression tests for bug fixes to prevent recurrence
3. Use existing test files as patterns (e.g., `usePty.test.ts` for hooks)
4. Mock Tauri APIs using the setup in `src/test/setup.ts`
5. Use `data-testid` attributes to query elements in tests, not CSS classes (Tailwind classes are brittle and change frequently)

## Commits and Releases

This project uses [Conventional Commits](https://www.conventionalcommits.org/) with release-please for automated versioning.

### Commit Message Format

```
<type>: <description>

[optional body]

[optional footer]
```

### Version Bump Rules

| Commit Type | Version Bump | Example |
|-------------|--------------|---------|
| `feat:` | Minor (0.1.0 → 0.2.0) | `feat: add dark mode toggle` |
| `fix:` | Patch (0.1.0 → 0.1.1) | `fix: resolve crash on startup` |
| `feat!:` or `BREAKING CHANGE:` | Major (0.1.0 → 1.0.0) | `feat!: change config format` |
| `docs:`, `chore:`, `refactor:`, `test:`, `style:` | No bump | `docs: update README` |

### Commit Types

- `feat` - New feature for the user
- `fix` - Bug fix for the user
- `docs` - Documentation only changes
- `style` - Formatting, missing semicolons, etc (no code change)
- `refactor` - Code change that neither fixes a bug nor adds a feature
- `perf` - Performance improvement
- `test` - Adding or updating tests
- `chore` - Maintenance tasks, dependency updates
- `build` - Changes to build system or external dependencies
- `ci` - CI configuration changes

### Release Process

1. Make commits using conventional commit format
2. Push to `main` branch
3. Release-please automatically creates/updates a release PR
4. Review and merge the release PR
5. GitHub Action builds and uploads binaries to the release

### Manual Version Sync

Version is defined in three places that release-please keeps in sync:
- `package.json` - `version` field
- `src-tauri/Cargo.toml` - `version` field
- `src-tauri/tauri.conf.json` - `version` field

## Logging

All logs (Rust backend + TypeScript frontend) are unified for easy debugging.

### Log Outputs

| Source | Terminal (stdout) | Log File | Browser DevTools |
|--------|-------------------|----------|------------------|
| Rust `log::info!()` | ❌ | ✅ | ✅ |
| Rust `eprintln!()` | ✅ (stderr) | ❌ | ❌ |
| TS `log.info()` | ✅ | ✅ | ✅ |
| TS `console.log()` | ❌ | ❌ | ✅ |

**Use `log.info()` for TypeScript logs** - they appear everywhere.

### Log Location

```bash
# Live tail during development
tail -f ~/Library/Logs/com.shellflow.desktop/shellflow.log

# Read full log
cat ~/Library/Logs/com.shellflow.desktop/shellflow.log
```

### Adding Logs

**Rust** - Use the `log` crate macros (goes to file + browser):
```rust
use log::{info, warn, error, debug, trace};

info!("[function_name] Starting operation...");
error!("[function_name] Failed: {}", e);
```

**TypeScript** - Use the `log` utility (goes to terminal + file + browser):
```typescript
import { log } from '../lib/log';

log.info('[ComponentName] Mounting with props:', props);
log.warn('[ComponentName] Unexpected state:', state);
log.error('[ComponentName] Failed to fetch:', error);
```

### Logging Guidelines

1. **Always prefix with context** - Use `[function_name]` or `[ComponentName]` prefix
2. **Log timing for operations** - Wrap slow operations with `Instant::now()` / `performance.now()`
3. **Log state transitions** - When important state changes, log before/after
4. **Log errors with context** - Include relevant IDs, paths, or parameters
5. **Use appropriate levels**:
   - `log.error` / `error!` - Failures that need attention
   - `log.warn` / `warn!` - Unexpected but recoverable situations
   - `log.info` / `info!` - Key operations and timing
   - `log.debug` / `debug!` - Detailed debugging (not shown by default)

### Performance Logging Pattern

```rust
let start = std::time::Instant::now();
// ... operation ...
info!("[operation_name] took {:?}", start.elapsed());
```

```typescript
const start = performance.now();
// ... operation ...
log.info(`[operationName] took ${performance.now() - start}ms`);
```

## Configuration

User config is stored at `~/.config/shellflow/config.jsonc`:

```jsonc
{
  "main": {
    "command": "claude",  // Command to run in main pane
    "fontFamily": "Menlo, Monaco, monospace",
    "fontSize": 13
  },
  "worktree": {
    "directory": "{{ repo_directory }}/.worktrees",
    "copy": {
      "gitignored": true,
      "except": [".claude"]
    }
  }
}
```

### Adding Configurable Options

When adding new configurable options, update all three files:
1. `src-tauri/src/config.rs` - Rust struct and defaults
2. `src-tauri/src/default_config.jsonc` - Default config with comments
3. `schemas/config.schema.json` - JSON Schema for validation

---
> Source: [shkm/shellflow](https://github.com/shkm/shellflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
