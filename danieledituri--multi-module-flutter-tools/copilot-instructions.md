## multi-module-flutter-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Architecture Overview

**Multi Module Flutter Tools** is a VS Code extension for running Flutter/Dart commands across multi-module workspaces. The architecture is:

### Core Layers

1. **Extension Entry** (`src/extension.ts`)
   - Registers all 15 commands in the command palette
   - Manages the output channel (streaming shell command output)
   - Orchestrates two main operation types: `runProjectOperation()` and `runWorkspaceOperation()`
   - Each operation tracks `OperationStats` (success/failure counts, duration, errors per module)

2. **UI Layer** (Webview Sidebar)
   - `MultiModuleViewProvider` + `MultiRepoViewProvider`: render sidebar sections (Cache, Packages, Git, Analysis, Run)
   - `BaseWebviewProvider` (base class): shared webview utilities, nonce generation, codicon loading
   - Commands are button clicks in the HTML sidebar → message posted to extension → command executed

3. **Shell Execution** (`runShellCommand()` in extension.ts)
   - Uses `child_process.spawn()` for streaming output (not buffered exec)
   - Respects VS Code `CancellationToken` — killing the process immediately on cancel
   - Applies FVM proxy if configured (`multiModuleFlutter.useFvm`)
   - Returns `{ ok: boolean; message?: string }`

4. **Project Discovery** (`src/repoDiscovery.ts`)
   - Scans workspace for `pubspec.yaml` files (Flutter modules)
   - Respects `multiModuleFlutter.maxDepth` and `multiModuleFlutter.excludeFolders` settings
   - Returns flat list of `ProjectInfo` (name, path)

5. **Notifications & Messages** (`src/notificationManager.ts`, `src/constants.ts`)
   - `NotificationManager`: shows VSCode popups (success/error/warning/info)
   - `constants.ts`: 26 error message categories + success templates (single source of truth)
   - Auto-categorizes completion notifications based on success/failure counts

6. **Type System** (`src/types.ts`)
   - `ProjectInfo`: `{ name, path }`
   - `OperationStats`: `{ successCount, failureCount, totalCount, durationMs, cancelled, errors }`
   - `CommandResult`, `NotificationConfig`: standardized across operations

### Command Flow Example
User clicks "Pub Get" → sidebar button posts message → extension receives `runCacheRepair` command → `runWorkspaceOperation()` called → for each workspace root, spawn `flutter pub get` → collect stats → notify completion with emoji (✅/⚠️/❌)

---

## Development Commands

### Build & Compilation
```bash
npm run compile        # TypeScript → dist/extension.js (includes esbuild minification)
npm run watch         # Continuous compile (watch mode)
npm run check-types   # TypeScript type checking (no emit)
npm run lint          # ESLint check
```

### Testing
```bash
npm test              # Run all 56 tests (Mocha via VSCode Test CLI)
npm run compile-tests # Compile tests only (tsc -p . --outDir out)
npm run watch-tests   # Watch test compilation
```

### Package & Release
```bash
npm run package       # Production build (check-types + lint + minify)
npm run vscode:prepublish  # Pre-publication hook (runs package)
```

### Single Test (if needed)
Edit the test file filter in `src/test/extension.test.ts` (e.g., `suite.only()` or `test.only()`) then run `npm test`. Or use:
```bash
npm run compile-tests && npx mocha out/test/extension.test.js --grep "test name pattern"
```

---

## Test Architecture (56 Tests, ~60% Coverage)

### Test File Structure
- `src/test/extension.test.ts`: 15 test suites, 56 passing tests
- `src/test/testUtils.ts`: mock infrastructure (MockChildProcess, MockFileSystem, MockGit) + test data

### Key Test Suites
1. **Command Registration** (2 tests): Validates all 15 commands are in constants
2. **Notifications & Messages** (3 tests): Error/success message availability
3. **Mock Infrastructure** (18 tests):
   - `MockChildProcess`: spawn() event simulation
   - `MockFileSystem`: fs.readFile/writeFile with ENOENT handling
   - `MockGit`: git stash list/pop operations
4. **Shell Command Execution** (4 tests): Success/failure/cancellation paths
5. **File Operations** (4 tests): pubspec.yaml parsing, path line detection
6. **Statistics & Error Handling** (9 tests): OperationStats tracking, error collection per module
7. **Progress & Cancellation** (4 tests): CancellationToken behavior, multiple handlers

### Coverage by Function
- `runShellCommand()`: ~70% (spawn success/failure, cancellation)
- `convertDependenciesToLocal()`: ~65% (file I/O, pubspec parsing)
- `popNamedStash()`: ~60% (git stash operations)
- Error handling paths: ~80%

### Running Specific Tests
```bash
npm test                    # All 56 tests
npm run compile-tests       # Just compile
npm run watch-tests         # Watch mode
```

---

## Key Implementation Patterns

### 1. Multi-Module Operation Pattern
Both `runProjectOperation()` and `runWorkspaceOperation()` follow this flow:
```typescript
const errors: Array<{ module: string; message: string }> = [];
let successCount = 0, errorCount = 0;
const startTime = Date.now();

await vscode.window.withProgress(..., async (progress, token) => {
  for (const item of items) {
    if (token.isCancellationRequested) break;
    progress.report({ message: item.name, increment });
    try {
      await action(item, token);
      successCount++;
    } catch (error) {
      errorCount++;
      errors.push({ module: item.name, message: error.message });
    }
  }
});

notificationManager.notifyCompletion(operationName, {
  successCount, failureCount: errorCount, totalCount: items.length,
  durationMs: Date.now() - startTime, cancelled, errors
});
```

### 2. Command Registration Pattern
All 15 commands follow the same template in `extension.ts`:
```typescript
const runCacheRepair = async () => {
  await runWorkspaceOperation("Cache Repair", async (root, token) => {
    await runShellCommand("flutter pub cache repair", root, output, token);
    await runShellCommand("dart pub cache repair", root, output, token);
  }, output);
};
context.subscriptions.push(vscode.commands.registerCommand(COMMANDS.CACHE_REPAIR, runCacheRepair));
```

### 3. Error Message Lookup
Always use `constants.ts` for user-facing messages:
```typescript
import { ERROR_MESSAGES, SUCCESS_MESSAGES } from "./constants";
output.appendLine(ERROR_MESSAGES.FILE_NOT_FOUND(filePath));
output.appendLine(SUCCESS_MESSAGES.OPERATION_COMPLETED("Pub Get", 3));
```

### 4. Streaming Shell Output
Never use `exec()` or buffering. Always use `spawn()`:
```typescript
const child = spawn("sh", ["-c", command], { cwd });
child.stdout.on("data", (chunk) => output.append(chunk.toString()));
child.stderr.on("data", (chunk) => output.append(chunk.toString()));
child.on("close", (code) => { /* handle exit code */ });
```

### 5. FVM Proxy Application
Before running any flutter/dart command:
```typescript
const proxied = applyFvmProxy(command); // Prepends "fvm" if useFvm setting is true
```

---

## Configuration

All settings are in `multiModuleFlutter.*` namespace:
- `scanNested` (boolean, default true): Recursively scan workspace
- `maxDepth` (number, default 2): Directory scan depth
- `excludeFolders` (array): Folders to skip (node_modules, .git, etc.)
- `uiScale` (string, default "x-large"): Button/icon size (compact → xx-large)
- `useFvm` (boolean, default false): Prefix flutter/dart with fvm

Get config with: `vscode.workspace.getConfiguration("multiModuleFlutter").get<T>(key, default)`

---

## Module Discovery

`getAllFlutterProjects()` (in `repoDiscovery.ts`):
- Recursively walks workspace folders up to `maxDepth`
- Yields paths where `pubspec.yaml` exists
- Excludes folders in `excludeFolders` list
- Called once per session; results are sorted alphabetically

---

## Output Channel

One global `OutputChannel` created at activation. All commands append live (non-buffered) output:
```typescript
output.appendLine(`$ flutter pub get`);
output.append(stdoutChunk);
output.appendLine("✅ Command executed successfully");
output.show(true); // Preserve focus on other panels
```

---

## Notable Files Not To Over-Refactor

- `viewProviderUtils.ts`: Contains `BaseWebviewProvider` (shared by both view providers) — changes cascade to both sidebars
- `multiRepoViewProvider.ts`: Mirror of MultiModuleViewProvider for multi-root workspaces; keep in sync when updating UI structure
- `constants.ts`: Single source of truth for all messages; 26 error categories + success templates — add new categories here, not inline

---

## Git & Branching

- Main development branch: `main`
- Published versions: Use semantic versioning (e.g., v1.3.0)
- Current version in `package.json`: 1.3.0
- Published to: VS Code Marketplace + OpenVSX Registry

---

## Type Safety & Linting

- TypeScript strict mode enabled (tsconfig.json)
- ESLint checks all imports and code style
- No ESLint config file (uses project defaults)
- Always use explicit types for function params and return values, especially for async operations

---

## Common Additions

### Adding a New Command
1. Add command ID to `src/constants.ts` (COMMANDS object)
2. Add command title to `src/constants.ts` (COMMAND_TITLES object)
3. In `src/extension.ts`, create handler and register with `context.subscriptions.push(vscode.commands.registerCommand(...))`
4. Add UI button to sidebar in `multiModuleViewProvider.ts` (and mirror in `multiRepoViewProvider.ts` if applicable)
5. Add message templates to `constants.ts` (ERROR_MESSAGES, SUCCESS_MESSAGES)
6. Add test case to `src/test/extension.test.ts`

### Adding Error Messages
Always add to `src/constants.ts` under the appropriate category:
```typescript
export const ERROR_MESSAGES = {
  // Git errors
  GIT_CHECKOUT_FAILED: (branch: string) => `Failed to checkout branch "${branch}"...`,
  // File errors
  FILE_NOT_FOUND: (path: string) => `File not found: ${path}...`,
  // ... etc
};
```

### Testing a New Function
Create mocks in `src/test/testUtils.ts`, then add test suite in `src/test/extension.test.ts`:
```typescript
suite('My New Feature', () => {
  test('does something', () => {
    const mock = createMockSpawn(0, false);
    assert.ok(mock.exitCode === 0);
  });
});
```

---

## Performance Notes

- Shell commands are long-running (5–60s typical) — always use cancellation tokens
- Workspace scanning happens once on activation — do not call `getAllFlutterProjects()` repeatedly
- Output channel is unbuffered — safe to append frequently
- Progress notifications are real-time in the UI
- Tests run in 299ms total (fast mock-based approach)

---

## Known Limitations & Tradeoffs

1. **No real git operations in tests** — mocked for speed; full integration tests would require a test repo
2. **No real filesystem operations in tests** — mocked; catching actual fs errors relies on integration testing
3. **Module discovery is path-based** (pubspec.yaml presence), not build system aware
4. **Output panel scrolls with output** (not always kept at bottom) — user can pin if desired
5. **FVM must be in PATH** if `useFvm` enabled

---
> Source: [DanieleDituri/multi-module-flutter-tools](https://github.com/DanieleDituri/multi-module-flutter-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
