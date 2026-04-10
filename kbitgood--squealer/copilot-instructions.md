## squealer

> This document provides instructions for AI coding agents working on the Squealer project.

# Squealer - Agent Instructions

This document provides instructions for AI coding agents working on the Squealer project.

## Project Documentation

Before starting any task, review the relevant documentation in the `docs/` folder:

| Document | When to Read |
|----------|--------------|
| `PRD.md` | Before implementing any feature - understand requirements and feature ordering |
| `TECHNICAL_SPEC.md` | Before writing Rust backend code - understand architecture, data models, APIs |
| `UI_SPEC.md` | Before writing React frontend code - understand layouts, components, interactions |
| `TEST_PLAN.md` | Before AND after implementing any feature - understand how to validate your work |

**Important**: Features in the PRD are ordered by build dependency. Earlier features must be completed and tested before later features can be properly validated. Follow this order.

---

## General Development Rules

### 0. Forbidden Tools

**NEVER use the Chrome DevTools MCP tools**, even if they are installed and available. The Chrome DevTools MCP is not compatible with Tauri webviews and will not work correctly. Always use the Tauri MCP tools instead for any UI testing or interaction.

Forbidden tools include any tool prefixed with `chrome-devtools_` such as:
- `chrome-devtools_click`
- `chrome-devtools_fill`
- `chrome-devtools_take_snapshot`
- `chrome-devtools_navigate_page`
- etc.

### 1. Read Before You Code

- Always read the relevant documentation sections before starting implementation
- If a task spans multiple areas (e.g., backend + frontend), read both specs
- Check the TEST_PLAN.md for the specific feature to understand acceptance criteria

### 2. Test Your Work

Every feature implementation must include:

1. **Unit tests** for new functions/components
2. **Integration tests** where specified in TEST_PLAN.md
3. **Manual E2E testing via Tauri MCP** - All E2E tests must be performed manually by the agent using the Tauri MCP tools (screenshots, execute_js, etc.). Do NOT use Playwright or any browser automation tools.

### 3. Regression Testing

**After completing any task**, you must:

1. Identify related features that could be affected by your changes
2. Run tests for those related features
3. Verify nothing is broken

**Regression testing guidelines:**

| If you changed... | Also test... |
|-------------------|--------------|
| Connection storage/config | Connection testing, SSH tunnel, schema browsing |
| SSH tunnel code | Connection testing, export with SSH |
| Schema/table browsing | Table selection, export configuration |
| Export engine | All export options, progress tracking, queue management |
| Progress tracking | Queue management, pause/resume |
| Export options (any) | Basic export, round-trip import verification |
| History storage | History loading, default settings behavior |
| UI components | Related E2E tests, keyboard shortcuts |
| Keychain integration | Connection storage, SSH config, security tests |

### 4. Don't Commit Without Review

**Never commit changes automatically.** The user will review changes in their IDE before committing. When you complete a task:

1. Explain what was changed
2. List the files modified
3. Describe how you tested it
4. Wait for the user to review and commit

---

## Test Environment

### Docker Test Environment

The project uses Docker for integration testing. Before running integration tests:

1. Ensure Docker Desktop is running
2. Start the test environment:
   ```bash
   docker-compose -f tests/docker/docker-compose.yml up -d
   ```
3. Wait for containers to be healthy
4. Run your tests
5. Tear down when done:
   ```bash
   docker-compose -f tests/docker/docker-compose.yml down
   ```

### Test Database

The test database includes:
- Multiple schemas (`test_db_full`, `test_db_empty`, `test_db_views_only`)
- Multiple users with different permissions
- 50+ tables with varied naming patterns
- Tables with various data types and sizes
- Unicode test data

See `tests/fixtures/seed.sql` for the complete test data setup.

---

## Feature Implementation Workflow

### Before Starting

1. Read the PRD section for the feature
2. Read the TECHNICAL_SPEC section for implementation details
3. Read the UI_SPEC section if there's a UI component
4. Read the TEST_PLAN section to understand validation criteria
5. Check if prerequisite features are complete

### During Implementation

1. Follow the architecture defined in TECHNICAL_SPEC.md
2. Match the UI designs in UI_SPEC.md
3. Write tests as you go (not just at the end)
4. Keep changes focused - don't scope creep

### After Implementation

1. Run all tests for the feature (unit, integration, E2E)
2. Identify and run regression tests for related features
3. Verify the feature works end-to-end manually if possible
4. Document any deviations from the spec (and why)
5. Report completion to user and wait for review

---

## Code Organization

### Rust Backend (`src-tauri/`)

```
src-tauri/
├── src/
│   ├── main.rs           # Tauri entry point
│   ├── commands/         # Tauri command handlers
│   ├── db/               # MySQL client, queries
│   ├── ssh/              # SSH tunnel management
│   ├── export/           # Export engine
│   ├── settings/         # App settings, history
│   └── keychain/         # macOS Keychain integration
├── Cargo.toml
└── tauri.conf.json
```

### React Frontend (`src/`)

```
src/
├── components/           # Reusable UI components
├── views/                # Full page/tab views
├── hooks/                # Custom React hooks
├── stores/               # Zustand stores (state management)
├── api/                  # Tauri command wrappers
├── utils/                # Utility functions
└── App.tsx
```

### Tests (`tests/`)

```
tests/
├── docker/
│   └── docker-compose.yml
├── fixtures/
│   ├── seed.sql
│   └── ssh-keys/
├── integration/          # Rust integration tests
├── e2e/                  # E2E test notes/checklists (manual testing via Tauri MCP)
└── unit/                 # Additional unit tests
```

---

## Common Pitfalls

### Don't Do This

- ❌ Skip reading documentation before implementing
- ❌ Implement features out of order (breaks testing)
- ❌ Forget to run regression tests
- ❌ Commit without user review
- ❌ Store passwords anywhere except macOS Keychain
- ❌ Use MB in UI for file splitting (should be GB)
- ❌ Forget to handle errors gracefully

### Do This

- ✅ Read relevant docs first
- ✅ Follow feature build order from PRD
- ✅ Write tests alongside code
- ✅ Run regression tests after changes
- ✅ Wait for user to commit
- ✅ Use Keychain for all credentials
- ✅ Handle all error cases with user-friendly messages

---

## Asking for Clarification

If something is ambiguous or unclear:

1. Check if the answer is in the documentation
2. If not, ask the user before making assumptions
3. Document any decisions made for future reference

It's better to ask than to implement something incorrectly.

---

## Logging Strategy

All code in this project must include comprehensive logging for debugging and manual testing. This is critical because Tauri apps cannot be easily automated with standard browser testing tools.

### Rust Backend Logging

The backend uses the `log` and `env_logger` crates. Logging is initialized in `main.rs`.

**Log Levels:**
- `error!` - Operation failures, exceptions
- `warn!` - Recoverable issues, unexpected but handled conditions
- `info!` - Key operation start/completion, state changes
- `debug!` - Detailed data, intermediate values (use for sensitive lengths, not values)

**Prefixes:** Use bracketed prefixes to identify the source module:
- `[CMD]` - Tauri command handlers (`src/commands/`)
- `[DB]` - Database operations (`src/db/`)
- `[SSH]` - SSH tunnel operations (`src/ssh/`)
- `[KEYCHAIN]` - Keychain operations (`src/keychain/`)
- `[EXPORT]` - Export engine (`src/export/`)
- `[SETTINGS]` - Settings/history (`src/settings/`)

**Example patterns:**

```rust
use log::{info, warn, error, debug};

// Command entry point
info!("[CMD] create_connection called - label: {}", config.label);
debug!("[CMD] create_connection config: {:?}", config);

// Success
info!("[CMD] create_connection success - id: {}", connection.id);

// Failure
error!("[CMD] create_connection failed for '{}': {}", config.label, e);

// Phase tracking for multi-step operations
info!("[DB] Phase 1: Establishing SSH tunnel to {}:{}", host, port);
info!("[DB] Phase 2: Connecting to MySQL...");

// Sensitive data - log presence/length, never the value
debug!("[KEYCHAIN] MySQL password retrieved successfully (length: {})", pwd.len());
```

**Running with different log levels:**

```bash
# Default (info level)
npm run tauri:dev

# Debug level for more detail
RUST_LOG=debug npm run tauri:dev

# Specific module only
RUST_LOG=squealer::ssh=debug npm run tauri:dev
```

### React Frontend Logging

Use `console.log`, `console.debug`, `console.error` with `[API]` or `[UI]` prefixes.

**Prefixes:**
- `[API]` - API wrapper functions (`src/api/`)
- `[UI]` - React component state/lifecycle (`src/views/`, `src/components/`)

**Example patterns:**

```typescript
// API call start
console.log('[API] createConnection: calling backend...', {
  label: params.config.label,
  host: params.config.mysql.host,
  hasSSH: !!params.config.ssh,
});

// API success
console.log('[API] createConnection: success, id:', result.id);
console.debug('[API] createConnection: result:', result);

// API failure
console.error('[API] createConnection: failed:', error);

// UI state changes
console.log('[UI] ConnectionsListView mounted, loading connections...');
console.log('[UI] Rendering connection list with', connections.length, 'items');
```

### What to Log

**Always log:**
1. Entry to Tauri commands (with key parameters)
2. Success/failure of operations (with IDs, counts)
3. Phase transitions in multi-step operations (SSH + MySQL)
4. State changes in React components (loading, error, success)
5. API calls start and completion

**Never log:**
1. Actual passwords, keys, or secrets
2. Full credential values (log length instead)
3. Large data structures in production (use `debug!` level)

### Viewing Logs

**Backend logs:** Appear in the terminal where `npm run tauri:dev` is running.

**Frontend logs:** Open the WebView DevTools:
- Right-click in the app window → "Inspect Element"
- Or use the keyboard shortcut (Cmd+Option+I on macOS)
- Navigate to Console tab

---

## Manual UI Testing with Tauri MCP

The Tauri MCP (Model Context Protocol) server provides tools for manually testing the running Tauri application. This is essential for verifying UI behavior since Tauri webviews don't work well with standard browser automation.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| `tauri-mcp_take_screenshot` | Capture the current state of the app window |
| `tauri-mcp_execute_js` | Run JavaScript in the webview context |
| `tauri-mcp_get_dom` | Retrieve the full HTML DOM (may timeout on complex pages) |
| `tauri-mcp_get_element_position` | Find element coordinates by selector |
| `tauri-mcp_simulate_mouse_movement` | Move/click at screen coordinates |

### Testing Workflow

1. **Take a screenshot first** to see the current app state
2. **Use `execute_js` to interact** with the UI (clicking buttons, filling forms)
3. **Take another screenshot** to verify the result
4. **Repeat** as needed to test the full flow

### Clicking Buttons and Interacting with Elements

The most reliable way to interact with UI elements is via JavaScript execution:

```javascript
// Find and click a button by its text content
const buttons = document.querySelectorAll('button');
for (const btn of buttons) {
  if (btn.textContent.trim() === 'ButtonText') {
    btn.click();
    break;
  }
}
```

For buttons that may appear multiple times (e.g., "Delete" in a list and in a modal), use additional selectors:

```javascript
// Click a button with specific class (e.g., in a modal)
const buttons = document.querySelectorAll('button');
for (const btn of buttons) {
  if (btn.textContent.trim() === 'Delete' && btn.classList.contains('bg-error')) {
    btn.click();
    break;
  }
}
```

### Important Tauri Webview Limitations

Be aware of these limitations when testing or writing code:

1. **`window.confirm()` doesn't work** - Native browser dialogs are blocked in Tauri webviews. Always use custom modal components for confirmations.

2. **`window.alert()` may not display** - Similar to confirm, use toast notifications or modals instead.

3. **`window.prompt()` doesn't work** - Use form inputs in modals instead.

4. **DOM queries may timeout** - The `get_dom` tool can timeout on complex pages. Prefer `execute_js` for targeted queries.

5. **Element position tools may timeout** - If `get_element_position` times out, use `execute_js` to click elements directly via `.click()`.

### Debugging Tips

1. **Add console.log statements** in your JavaScript to trace execution:
   ```javascript
   console.log('Found button, clicking...');
   btn.click();
   ```

2. **Check for React state issues** - If UI doesn't update after an action, the issue may be in React state management, not the click itself.

3. **Use screenshots liberally** - Take screenshots before and after each action to verify state changes.

4. **Test error paths** - Verify that error modals and messages display correctly.

### Common Patterns

**Testing a button opens a modal:**
```javascript
// Click button
document.querySelector('button').click();
// Then take screenshot to verify modal appeared
```

**Testing form submission:**
```javascript
// Fill form fields
document.querySelector('input[name="label"]').value = 'Test';
// Trigger React's onChange if needed
document.querySelector('input[name="label"]').dispatchEvent(
  new Event('input', { bubbles: true })
);
// Click submit
document.querySelector('button[type="submit"]').click();
```

**Closing modals:**
```javascript
// Click Cancel or Close button
const buttons = document.querySelectorAll('button');
for (const btn of buttons) {
  if (btn.textContent.trim() === 'Cancel') {
    btn.click();
    break;
  }
}
```

---

## Quick Reference: Test Commands

```bash
# Rust unit tests
cargo test

# Rust integration tests (requires Docker)
cargo test --features integration

# React unit tests
npm test

# E2E tests - perform manually using Tauri MCP tools
# (Do NOT use Playwright - use tauri-mcp_take_screenshot, tauri-mcp_execute_js, etc.)

# Start test Docker environment
docker-compose -f tests/docker/docker-compose.yml up -d

# Stop test Docker environment
docker-compose -f tests/docker/docker-compose.yml down

# Reset test data
docker-compose -f tests/docker/docker-compose.yml exec mysql mysql -u root -p < tests/fixtures/seed.sql
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kbitgood)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kbitgood)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
