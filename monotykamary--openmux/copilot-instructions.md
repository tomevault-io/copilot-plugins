## openmux

> This file provides guidance to coding agents working with the openmux repository.

# AGENTS.md

This file provides guidance to coding agents working with the openmux repository.

## Build and Run Commands

```bash
bun install           # Install dependencies
bun start             # Build and run the terminal multiplexer
bun dev               # Run with watch mode (--watch)
bun run typecheck     # Type check without emitting
bun run lint          # Lint Effect code (effect-language-service)
bun run build         # Build standalone binary (./scripts/build.sh)
bun run build:release # Build optimized binary
bun run install:local # Build and install locally
bun run test          # Run bun:test TS + Zig + Ghostty VT tests
bun run test:ts       # Run bun:test suite only
bun run test:pty      # Run Zig PTY tests only
bun run test:ghostty-vt # Run Ghostty VT Zig tests
bun run test:watch    # Run bun:test in watch mode
bun run check:circular # Detect circular deps in src/
```

## libghostty-vt Notes

- Ghostty is tracked as a submodule in `vendor/ghostty`.
- Wrapper library lives in `native/zig-ghostty-wrapper` and exports the C API.
- `scripts/update-ghostty-vt.sh` updates the submodule only (no patches).
- `scripts/update-libgit2.sh` updates `vendor/libgit2` directly (no patches).
- `scripts/build.sh` builds `libghostty-vt` via the wrapper before bundling.

## Technology Stack

- **Bun** - Runtime, package manager, and test runner (`bun:test`)
- **OpenTUI** - Terminal UI library with SolidJS reconciler (@opentui/core, @opentui/solid)
- **SolidJS** - Reactive UI framework via OpenTUI's SolidJS reconciler
- **zig-pty** - PTY support for shell process management (pure Zig implementation)
- **libghostty-vt** - Native terminal emulator (VT parsing/state)
- **errore** - Type-safe error handling with Golang-style returns (replaces heavy Effect.ts)

## Architecture Overview

openmux is a terminal multiplexer with a master-stack tiling layout (Zellij-style). The UI is SolidJS components rendered to the terminal via OpenTUI, with PTYs managed in Zig and emulated via native libghostty-vt.

### Entry Points

- `src/index.tsx` - CLI entry, renderer setup
- `src/shim/main.ts` - Shim server entry (background detach/attach)
- `src/App.tsx` - Provider tree and top-level app wiring

### Core Data Flow

```
Keyboard Input → KeyboardContext → Layout/Terminal actions
                                ↓
                    Master-stack layout calculation
                                ↓
                       PaneContainer/TerminalView
PTY Data → zig-pty → GhosttyVTEmulator (libghostty-vt) → Shim protocol → TerminalContext
                                ↓
                   TerminalView + AggregateView preview
Session data → SessionContext → disk persistence → SessionBridge (Layout/Terminal)
Title updates → TitleContext → StatusBar/window title
```

Note: In shim server mode, PTY data stays local; in shim client mode, it flows through the shim protocol.

Detach/attach uses a single-client lock; new clients steal the lock and detach the previous client.

### Context Hierarchy (src/App.tsx)

```tsx
ThemeProvider              // Styling/theming
  └── LayoutProvider       // Workspace/pane state (reducer pattern)
        └── KeyboardProvider    // Prefix mode, key state
              └── TitleProvider     // Terminal title updates
                    └── TerminalProvider   // PTY lifecycle, terminal state
                          └── SelectionProvider  // Text selection state
                                └── SearchProvider    // Terminal search
                                      └── SessionBridge   // SessionProvider wiring
                                            └── AggregateViewProvider  // Cross-workspace overlay
                                                  └── AppContent
```

### Key Modules

**Layout and session state (src/core/)**
- `types.ts`, `config.ts` - Core types and defaults
- `operations/master-stack-layout.ts` - Layout calculation
- `operations/layout-actions/` - Pane/workspace actions
- `operations/session-actions/` - Session restore/save helpers
- `workspace-utils.ts`, `coordinates/`, `scroll-utils.ts`, `keyboard-utils.ts`

**Terminal layer (src/terminal/)**
- `ghostty-vt/`, `emulator-utils/` - native libghostty-vt bindings + shared emulator helpers
- `emulator-interface.ts` - ITerminalEmulator abstraction
- `input-handler.ts`, `sync-mode-parser.ts` - Input/escape handling
- `title-parser.ts`, `terminal-query-passthrough/` - Title/query parsing
- `paste-intercepting-stdin.ts`, `focused-pty-registry.ts`

**Shim / detach (src/shim/)**
- `main.ts`, `server.ts` - Shim server + RPC handling
- `client/` - Shim client connection, PTY state cache, detach handling

**UI components (src/components/)**
- `PaneContainer.tsx`, `Pane.tsx`, `TerminalView.tsx` - Pane rendering
- `AggregateView.tsx`, `SessionPicker.tsx`, `SearchOverlay.tsx`
- `StatusBar.tsx`, `CopyNotification.tsx`

**SolidJS contexts (src/contexts/)**
- `LayoutContext.tsx`, `TerminalContext.tsx`, `KeyboardContext.tsx`
- `SelectionContext.tsx`, `SearchContext.tsx`, `SessionContext.tsx`
- `ThemeContext.tsx`, `TitleContext.tsx`, `AggregateViewContext.tsx`

**Effect module (src/effect/)**
- `errors.ts` - Domain-specific tagged errors using errore
- `resources.ts` - ResourceStack for cleanup with `using`/`defer`
- `services/` - Service implementations (PTY, Session, etc.)
- `models.ts`, `types.ts` - Type definitions
- `bridge/` - SolidJS/Effect service bridge

## Error Handling Patterns (Golang-Style with errore)

We use the [`errore`](https://www.npmjs.com/package/errore) library for type-safe, Golang-style error handling instead of throwing exceptions.

### Core Principle

Functions return `Result | Error` unions. Callers explicitly check for errors using `instanceof`. This makes all error paths explicit and type-safe.

```typescript
import { tryAsync, tryFn as trySync } from 'errore';
import { createTaggedError } from 'errore';

// Define tagged errors with template messages
export class SessionStorageError extends createTaggedError({
  name: 'SessionStorageError',
  message: 'Session storage $operation failed for $path: $reason',
}) {}

// Async operations return unions: Promise<Result | Error>
async function loadSession(id: string): Promise<SessionData | SessionStorageError> {
  const result = await tryAsync<SessionData, SessionStorageError>({
    try: () => fs.readFile(getPath(id), 'utf8').then(JSON.parse),
    catch: (e) => new SessionStorageError({ operation: 'read', path: id, reason: String(e) }),
  });
  
  // Golang-style: if err != nil { return err }
  if (result instanceof SessionStorageError) {
    return result;
  }
  
  return validateSession(result);
}

// Sync operations work the same way
function parseConfig(path: string): Config | ConfigParseError {
  const result = trySync<Config, ConfigParseError>({
    try: () => TOML.parse(fs.readFileSync(path, 'utf8')) as Config,
    catch: (e) => new ConfigParseError({ path, reason: String(e) }),
  });
  
  if (result instanceof ConfigParseError) {
    console.warn('Failed to parse config:', result);
    return DEFAULT_CONFIG;
  }
  
  return result;
}
```

### Error Definitions

All domain errors are defined in `src/effect/errors.ts`:

```typescript
import { createTaggedError } from 'errore';

// PTY errors
export class PtySpawnError extends createTaggedError({
  name: 'PtySpawnError',
  message: 'Failed to spawn PTY shell $shell in $cwd: $reason',
}) {}

export class PtyNotFoundError extends createTaggedError({
  name: 'PtyNotFoundError',
  message: 'PTY session $ptyId not found',
}) {}

// Session errors
export class SessionNotFoundError extends createTaggedError({
  name: 'SessionNotFoundError',
  message: 'Session $sessionId not found',
}) {}

export class SessionCorruptedError extends createTaggedError({
  name: 'SessionCorruptedError',
  message: 'Session $sessionId is corrupted: $reason',
}) {}

// Union types for error handling
export type PtyError = PtySpawnError | PtyNotFoundError | PtyCwdError;
export type SessionError = SessionNotFoundError | SessionCorruptedError | SessionStorageError;
```

### Error Propagation

Errors are values that propagate explicitly up the call chain:

```typescript
async function initializeAndRender(): Promise<void | StartupError> {
  const services = await initializeServices();
  if (services instanceof Error) {
    return new StartupError({ reason: `Failed to initialize: ${services.message}` });
  }
  
  const renderResult = await tryAsync<void, StartupError>({
    try: () => render(() => <App />),
    catch: (e) => new StartupError({ reason: String(e) }),
  });
  
  // Return error up the chain
  if (renderResult instanceof StartupError) {
    return renderResult;
  }
}

// Top-level handling
const result = await initializeAndRender();
if (result instanceof StartupError) {
  console.error('Failed to start:', result);
  process.exit(1);
}
```

### Benefits

- **Type safety**: Return types explicitly include possible errors
- **No hidden throws**: Every error path is visible in the code
- **Explicit propagation**: Errors must be explicitly returned (or handled)
- **No stack unwinding**: Errors are values, not exceptions
- **Pattern matching**: `instanceof` checks work reliably across modules

## Resource Management with `using` and `defer`

We use TypeScript's Explicit Resource Management (`using` keyword) combined with `AsyncDisposableStack` from errore for automatic, guaranteed cleanup.

### The `using` Keyword

The `using` declaration (TypeScript 5.2+) automatically calls `[Symbol.dispose]` or `[Symbol.asyncDispose]` when the variable goes out of scope — even if an error is thrown.

```typescript
// Synchronous disposal
using file = fs.openSync('/tmp/data.txt', 'w');
fs.writeSync(file, 'data');
// file is automatically closed here

// Asynchronous disposal (what we use most)
async function fetchWithCleanup() {
  await using conn = await createConnection();
  await conn.send(data);
  // conn.disconnect() called automatically via asyncDispose
}
```

### ResourceStack: Golang-like defer

Our `ResourceStack` class (in `src/effect/resources.ts`) wraps `AsyncDisposableStack` with typed error handling and convenience methods:

```typescript
import { ResourceStack } from '../effect/resources';

async function processPty(ptyId: string): Promise<void | PtyError> {
  await using resources = new ResourceStack();
  
  // Defer cleanup (like Go's defer) - runs LIFO on scope exit
  const tempFile = await createTempFile();
  resources.defer(() => fs.unlink(tempFile));
  
  // Register subscriptions for auto-cleanup
  const unsub = subscribeToUpdates(ptyId, handleUpdate);
  resources.registerSubscription(unsub);
  
  // Register timers/intervals
  const timeout = setTimeout(() => cancelOperation(), 5000);
  resources.registerTimer(timeout);
  
  // Register AbortController
  const controller = new AbortController();
  resources.registerAbortController(controller);
  
  // Defer with error logging (cleanup failures don't block other cleanups)
  resources.deferSafe(() => {
    state.activeProcesses.delete(ptyId);
  });
  
  // Do work...
  const result = await doWork(tempFile, controller.signal);
  
  // All cleanup runs automatically when function exits
  return result;
}
```

### Guard Classes with AsyncDisposable

Common pattern for state guards that reset flags on exit:

```typescript
/** AsyncDisposable guard for refresh state flags */
class RefreshGuard implements AsyncDisposable {
  constructor(
    private state: RefreshState,
    private key: keyof RefreshState
  ) {
    this.state[this.key] = true;
  }

  async [Symbol.asyncDispose](): Promise<void> {
    this.state[this.key] = false;
  }
}

// Usage: automatically resets refreshInProgress when done
async function refreshData() {
  if (refreshState.refreshInProgress) return;
  await using _guard = new RefreshGuard(refreshState, 'refreshInProgress');
  
  // Do work while flag is true...
  await fetchData();
  
  // Flag automatically reset here, even if fetchData() throws
}
```

### Inline AsyncDisposable Objects

For quick cleanup without defining a class:

```typescript
await using cleanup = {
  [Symbol.asyncDispose]: async () => {
    client.off('data', handleData);
    client.off('close', handleClose);
    client.off('end', handleClose);
  },
};
```

### ResourceStack API Reference

```typescript
class ResourceStack extends AsyncDisposableStack {
  // Basic defer (LIFO order - last deferred runs first)
  defer(cleanup: () => void | Promise<void>): void;
  
  // Defer with error logging (cleanup errors don't stop other cleanups)
  deferSafe(cleanup: () => void | Promise<void>): void;
  
  // Defer multiple at once
  deferAll(...cleanups: Array<() => void | Promise<void>>): void;
  
  // Register common resource types
  registerTimer(timer: ReturnType<typeof setTimeout>): void;
  registerInterval(interval: ReturnType<typeof setInterval>): void;
  registerAbortController(controller: AbortController): void;
  registerSubscription(unsubscribe: () => void): void;
  registerEventListener(emitter, event, handler): void;
  registerDisposable<T extends AsyncDisposable>(resource: T): T;
}
```

### When to Use

| Pattern | Use Case |
|---------|----------|
| `await using resources = new ResourceStack()` | Multiple cleanup operations needed |
| `await using guard = new SomeGuard()` | State flag management (prevents re-entrant calls) |
| `await using _ = { [Symbol.asyncDispose]: ... }` | One-off inline cleanup |
| `using file = openSync(...)` | Synchronous resources (rare in our codebase) |

### Example: Session Operations

```typescript
async function handleSessionSwitch(id: string): Promise<void> {
  // Guaranteed cleanup: always close picker when function exits
  await using resources = new ResourceStack();
  resources.defer(() => {
    dispatch({ type: 'CLOSE_SESSION_PICKER' });
  });
  
  // Prevent concurrent switching
  await using _guard = new SwitchingGuard(dispatch, true);
  
  const result = await switchToSession(id);
  if (result instanceof SessionError) {
    console.error('Failed to switch:', result.message);
    return;
  }
  
  // picker closed and guard reset automatically
}
```

## SolidJS Reactivity Patterns

Contexts expose values via object properties. Understanding what's safe to destructure is critical:

**Safe to destructure (action functions):**
```tsx
const { newPane, closePane, focusPane } = useLayout();  // Plain functions
const { createPTY, writeToPTY } = useTerminal();        // Plain functions
```

**Safe to destructure (store proxy - accessing properties IS reactive):**
```tsx
const { state } = useLayout();
state.workspaces;           // Reactive - store proxy tracks access
state.activeWorkspaceId;    // Reactive
```

**NOT safe to destructure (computed getters - call signal/memo inside):**
```tsx
// DON'T: Destructuring calls the getter once, loses reactivity
const { activeWorkspace, panes, isInitialized } = useLayout();

// DO: Access via context object to call getter at access time
const layout = useLayout();
layout.activeWorkspace;     // Calls getter each time, stays reactive
layout.panes;               // Calls getter each time, stays reactive
```

**Computed getters by context (access via context object, don't destructure):**
- `LayoutContext`: `activeWorkspace`, `panes`, `paneCount`, `populatedWorkspaces`, `layoutVersion`
- `TerminalContext`: `isInitialized`
- `SessionContext`: `filteredSessions`
- `SelectionContext`: `selectionVersion`, `copyNotification`
- `SearchContext`: `searchState`, `searchVersion`

## Layout Modes

Each workspace has a `layoutMode` that determines pane arrangement:
- **vertical**: Main pane left, stack panes split vertically on right
- **horizontal**: Main pane top, stack panes split horizontally on bottom
- **stacked**: Main pane left, stack panes tabbed on right (only active visible)

## Workspaces and Sessions

- Workspaces are indexed 1-9 and maintain separate layout state.
- Sessions persist to `~/.config/openmux/sessions/` and are coordinated by `SessionContext` and `SessionBridge`.

## Code Style Guidelines

### Comments

**Avoid decorative comment sections.** Do not use patterns like:

```typescript
// ============================================================================
// Section Title
// ============================================================================
```

Instead, use:
- JSDoc comments for documentation: `/** Brief description */`
- Simple inline comments when necessary: `// Brief explanation`
- Group related functions with a single comment or whitespace, not decorative borders
- Session restores reuse existing PTYs when possible, otherwise new PTYs are created from stored CWDs.

---
> Source: [monotykamary/openmux](https://github.com/monotykamary/openmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
