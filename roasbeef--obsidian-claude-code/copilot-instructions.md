## obsidian-claude-code

> Obsidian plugin that embeds Claude AI as an assistant using the Claude Agent SDK. Features built-in tools, skill loading, and custom Obsidian-specific tools via SDK MCP server.

# obsidian-claude-code

## Project Overview
Obsidian plugin that embeds Claude AI as an assistant using the Claude Agent SDK. Features built-in tools, skill loading, and custom Obsidian-specific tools via SDK MCP server.

## Development Commands
```bash
# Install dependencies (use bun, not npm)
bun install

# Development mode with watch
bun run dev

# Production build
bun run build

# Type check only
bun run typecheck

# Run all checks (typecheck + lint + test)
make check

# Quick dev check (typecheck + unit tests only)
make quick
```

## Package Manager
**Use `bun` for all package management operations:**
- `bun install` - Install dependencies
- `bun add <package>` - Add dependency
- `bun add -d <package>` - Add dev dependency
- `bun remove <package>` - Remove dependency

We use Bun for package management but esbuild for bundling (Obsidian's standard approach).

## Project Structure
```
src/
├── main.ts                    # Plugin entry point
├── types.ts                   # TypeScript types
├── interfaces/                # DI interfaces for testability
│   ├── IVaultAdapter.ts       # File system abstraction
│   ├── IConversationStorage.ts # Persistence abstraction
│   └── ILogger.ts             # Logging abstraction
├── utils/
│   └── Logger.ts              # File-based debug logging
├── agent/
│   ├── AgentController.ts     # Claude Agent SDK query() orchestration
│   ├── ConversationManager.ts # Session/history persistence
│   └── ObsidianMcpServer.ts   # Custom Obsidian tools via SDK MCP
├── views/
│   ├── ChatView.ts            # Main sidebar (ItemView)
│   ├── MessageList.ts         # Message rendering
│   ├── MessageRenderer.ts     # Individual message
│   ├── ToolCallDisplay.ts     # Tool call UI
│   ├── ChatInput.ts           # Input with autocomplete
│   ├── AutocompletePopup.ts   # Command/file suggestions
│   └── ConversationHistoryModal.ts # History browser
└── settings/
    └── SettingsTab.ts         # Settings UI

tests/
├── unit/                      # Pure function tests
├── integration/               # Tests with mocked Obsidian
├── property/                  # Property-based tests (fast-check)
├── mocks/
│   ├── obsidian/              # Obsidian API mocks
│   └── claude-sdk/            # Claude SDK mocks
├── helpers/                   # Test utilities
└── setup.ts                   # Global test setup
```

## Testing in Obsidian
Symlink plugin to vault for hot-reload testing:
```bash
ln -s /Users/roasbeef/codez/obsidian-claude-code \
      /path/to/vault/.obsidian/plugins/obsidian-claude-code
```

Then in Obsidian: Settings > Community Plugins > Reload

## Key Architecture Decisions
- **Desktop-only** - Requires Node.js for Claude Agent SDK
- Uses `@anthropic-ai/claude-agent-sdk` with `query()` function
- Built-in tools (Read, Write, Bash, Grep, etc.) from SDK presets
- Skills auto-load from `vault/.claude/skills/` via `settingSources: ['project']`
- Custom Obsidian tools via SDK MCP server (`ObsidianMcpServer.ts`)
- Supports Claude Max subscription via `CLAUDE_CODE_OAUTH_TOKEN` env var
- Conversations stored in `.obsidian-claude-code/` in vault
- Right sidebar ItemView (Cursor-style chat)
- Debug logs written to `~/.obsidian-claude-code/debug.log`

## Electron/Obsidian Environment Considerations

### Claude CLI Path Resolution
The Agent SDK internally spawns the Claude Code CLI. In bundled Electron environments (like Obsidian), `import.meta.url` doesn't work, so the SDK can't auto-find the CLI.

**Solution**: Explicitly provide `pathToClaudeCodeExecutable` option:
```typescript
query({
  prompt: content,
  options: {
    pathToClaudeCodeExecutable: "/path/to/claude",
    // ...
  }
})
```

The plugin auto-detects Claude in common locations:
- `~/.nvm/versions/node/*/bin/claude`
- `/usr/local/bin/claude`
- `/opt/homebrew/bin/claude`

### PATH for Node.js
The Claude CLI is a Node.js script (`#!/usr/bin/env node`). Obsidian's Electron doesn't have nvm's PATH. The plugin adds the Claude bin directory to `env.PATH` when calling `query()`.

### Model Selection
Use simplified model names (not full IDs):
- `"sonnet"` - Claude Sonnet 4 (faster)
- `"opus"` - Claude Opus 4.5 (more capable)
- `"haiku"` - Claude Haiku (fastest)

### Obsidian File API
Use `vault.adapter.exists()` and `vault.adapter.write()` instead of `vault.getAbstractFileByPath()` for more reliable file operations. Obsidian's internal caching can cause race conditions with `vault.create()`.

## Dependencies
- `obsidian` - Obsidian API
- `@anthropic-ai/claude-agent-sdk` - Claude Agent SDK
- `zod` - Schema validation for SDK MCP tools

## Obsidian-Specific Tools (SDK MCP Server)
The plugin exposes these Obsidian-specific tools to Claude:
- `mcp__obsidian__open_file` - Open file in Obsidian view
- `mcp__obsidian__execute_command` - Run Obsidian command
- `mcp__obsidian__show_notice` - Display notification
- `mcp__obsidian__get_active_file` - Get current file info
- `mcp__obsidian__rebuild_vault_index` - Trigger vault-search index rebuild
- `mcp__obsidian__list_commands` - Discover available commands
- `mcp__obsidian__create_note` - Create new notes
- `mcp__obsidian__reveal_in_explorer` - Show in file explorer
- `mcp__obsidian__get_vault_stats` - Vault statistics
- `mcp__obsidian__get_recent_files` - Recently modified files

## Authentication
The plugin supports three authentication methods:
1. **API Key in Settings** - Enter key in plugin settings
2. **ANTHROPIC_API_KEY env var** - Set in your shell environment
3. **Claude Max subscription** - Run `claude setup-token` to create `CLAUDE_CODE_OAUTH_TOKEN`

For GUI apps like Obsidian to inherit env vars on macOS:
```bash
launchctl setenv CLAUDE_CODE_OAUTH_TOKEN "$(echo $CLAUDE_CODE_OAUTH_TOKEN)"
```

## Debugging
Debug logs are written to `~/.obsidian-claude-code/debug.log`. Tail them during development:
```bash
tail -f ~/.obsidian-claude-code/debug.log
```

Key log components:
- `[Plugin]` - Plugin lifecycle
- `[ChatView]` - UI and message handling
- `[ChatInput]` - Input and key events
- `[AgentController]` - SDK query and tool execution
- `[ConversationManager]` - Persistence

## Coding Conventions
- Use complete sentences ending with periods for comments.
- Follow Obsidian plugin patterns (ItemView, PluginSettingTab, etc.)
- CSS classes prefixed with `claude-code-`
- Log important events with the Logger utility

## Testing

### Running Tests
```bash
# Run all tests
make test

# Run tests with watch mode
make test-watch

# Run specific test suites
make test-unit        # Unit tests only
make test-integration # Integration tests only
make test-property    # Property-based tests only

# Run with coverage report
make coverage
make coverage-report  # Opens HTML report in browser
```

### Test Structure
```
tests/
├── unit/           # Pure function tests (no Obsidian deps)
├── integration/    # Tests with mocked Obsidian APIs
├── property/       # Property-based tests with fast-check
├── mocks/
│   ├── obsidian/   # App, Vault, ItemView, Modal mocks
│   └── claude-sdk/ # SDK query() mock
├── fixtures/       # Test data (conversations, messages)
└── helpers/        # Test utilities, factories
```

### Writing Tests
- Use Vitest for all tests (`describe`, `it`, `expect`).
- Mock Obsidian APIs using mocks in `tests/mocks/obsidian/`.
- Use factories in `tests/helpers/factories.ts` for test data.
- Property tests go in `.prop.test.ts` files.
- Target 85%+ coverage for all new code.

### Mocking Obsidian
```typescript
import { createMockApp, createMockVault } from "../mocks/obsidian";

const mockApp = createMockApp();
const mockVault = createMockVault();

// Configure mock behavior.
mockVault.getMarkdownFiles.mockReturnValue([...]);
```

### Property-Based Testing
Use fast-check for invariant testing:
```typescript
import fc from "fast-check";

fc.assert(
  fc.property(fc.string(), (input) => {
    const result = myFunction(input);
    expect(result).toBeDefined();
  })
);
```

### Pure Functions to Test
Key pure functions suitable for unit testing:
- `classifyError()` in AgentController.ts - Error type classification.
- `generateId()` - Unique ID generation.
- `generateTitle()` - Title truncation from content.

## Linting

### Running Linter
```bash
make lint       # Check for issues
make lint-fix   # Auto-fix issues
```

### ESLint Rules
- Strict TypeScript rules enabled (type-aware).
- Import ordering enforced (builtin → external → internal).
- No floating promises allowed.
- Strict boolean expressions required.
- Unused vars must be prefixed with `_`.

## CI/CD

### GitHub Actions
CI runs on every push and PR to main/develop:
1. **typecheck** - TypeScript type checking.
2. **lint** - ESLint checks.
3. **test** - Run tests with coverage.
4. **build** - Verify production build.

### Coverage Requirements
- Target: 85% line coverage.
- Coverage reports uploaded to Codecov.
- CI warns if coverage drops below threshold.

### Running CI Locally
```bash
make ci  # Run full CI pipeline locally
```

## Contributing

### Before Opening a PR
1. Run `make check` - All checks must pass.
2. Add tests for new functionality.
3. Ensure coverage doesn't decrease.
4. Update CLAUDE.md if adding new features.

### Code Review Expectations
- Tests for new code paths.
- No decrease in coverage.
- Follow existing patterns.
- Clear commit messages.

## Dependency Injection

### Interfaces for Testability
Located in `src/interfaces/`:
- `IVaultAdapter` - Abstract file system operations.
- `IConversationStorage` - Abstract conversation persistence.
- `ILogger` - Abstract logging.

### Usage Pattern
```typescript
class MyClass {
  constructor(
    plugin: ClaudeCodePlugin,
    deps?: {
      vaultAdapter?: IVaultAdapter;
      logger?: ILogger;
    }
  ) {
    this.vault = deps?.vaultAdapter ?? new ObsidianVaultAdapter(plugin.app.vault);
    this.logger = deps?.logger ?? defaultLogger;
  }
}
```

This allows injecting mocks in tests without modifying production code.

---
> Source: [Roasbeef/obsidian-claude-code](https://github.com/Roasbeef/obsidian-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
