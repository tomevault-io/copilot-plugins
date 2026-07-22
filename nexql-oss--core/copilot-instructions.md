## core

> VS Code extension for PostgreSQL database management featuring interactive SQL notebooks, real-time dashboard, AI chat assistant, and tree-based database explorer. Uses `pg` client for PostgreSQL connections.

# PostgreSQL Explorer - Copilot Instructions

## Project Overview
VS Code extension for PostgreSQL database management featuring interactive SQL notebooks, real-time dashboard, AI chat assistant, and tree-based database explorer. Uses `pg` client for PostgreSQL connections.

## Architecture

### Core Components
- **`src/extension.ts`**: Entry point - initializes services (singletons), registers all commands, providers, and tree views
- **`src/services/ConnectionManager.ts`**: Singleton managing PostgreSQL connections with connection pooling keyed by `{connectionId}:{database}`
- **`src/services/SecretStorageService.ts`**: Singleton wrapping VS Code SecretStorage for credential management
- **`src/providers/DatabaseTreeProvider.ts`**: Tree view provider showing connections → databases → schemas → objects hierarchy
- **`src/providers/NotebookKernel.ts`**: Executes SQL cells in `.pgsql` notebooks, provides SQL completions
- **`src/providers/ChatViewProvider.ts`**: AI chat assistant using modular services from `src/providers/chat/`

### Command Pattern
Commands live in `src/commands/{domain}.ts` with SQL templates extracted to `src/commands/sql/{domain}.ts`:
```typescript
// src/commands/tables.ts - uses NotebookBuilder pattern
import { TableSQL } from './sql';
await new NotebookBuilder(metadata)
    .addMarkdown(MarkdownUtils.header('...') + MarkdownUtils.infoBox('...'))
    .addSql(TableSQL.delete(schema, table))
    .show();
```

### Helper Utilities (`src/commands/helper.ts`)
- `getDatabaseConnection(item)` - validates item, gets connection with password, returns `{connection, client, metadata}`
- `NotebookBuilder` - fluent API for creating notebooks with `.addMarkdown()`, `.addSql()`, `.show()`
- `MarkdownUtils` - `header()`, `infoBox()`, `warningBox()`, `dangerBox()`, `operationsTable()`
- `ErrorHandlers.handleCommandError(err, action)` - standardized error handling

### Styling System (`src/common/`)
- **`htmlStyles.ts`**: CSS variables, `MarkdownBuilder` for consistent markdown formatting
- **`notebookTemplates.ts`**: `NotebookCellBuilder` and `CommonNotebookTemplates` for reusable notebook patterns
- **`rendererUtils.ts`**: Webview component builders using VS Code theme variables

## Key Patterns

### Singleton Services
```typescript
// Always access via getInstance()
ConnectionManager.getInstance().getConnection(config);
SecretStorageService.getInstance().getPassword(connectionId);
```

### Tree Item Structure
`DatabaseTreeItem` has: `type` (connection|database|schema|table|column|etc), `connectionId`, `databaseName`, `schema`, `label`

### Notebook Metadata
Notebooks store `PostgresMetadata` with `connectionId`, `databaseName`, `host`, `port`, `username`, `password`

## Development Workflow

```bash
npm run watch          # Development - auto-recompile TypeScript
npm run compile        # One-time build
npm run test           # Unit tests with Mocha + Chai + Sinon
npm run coverage       # Tests with coverage report
F5                     # Launch Extension Development Host
```

### Testing
Tests in `src/test/unit/` mock VS Code API via `src/test/unit/mocks/vscode.ts`. Use `module-alias` in `src/test/setup.ts`.

### Debug Output
```typescript
import { outputChannel } from './extension';
outputChannel.appendLine('Debug message');
```

## File Organization

| Path | Purpose |
|------|---------|
| `src/commands/{domain}.ts` | Command implementations (tables, views, functions, etc.) |
| `src/commands/sql/{domain}.ts` | SQL template functions - pure, no VS Code deps |
| `src/providers/` | VS Code providers (tree, notebook kernel, chat, completions) |
| `src/providers/chat/` | Modular chat services (AiService, DbObjectService, SessionService) |
| `src/services/` | Business logic singletons |
| `src/common/` | Shared types, styles, templates |
| `src/dashboard/` | Dashboard webview components |

## Adding New Features

### New Database Object Command
1. Add SQL templates in `src/commands/sql/{object}.ts`
2. Implement command in `src/commands/{object}.ts` using `NotebookBuilder`
3. Register command in `src/extension.ts` and `package.json` contributes.commands
4. Add tree view context menu in `package.json` contributes.menus

### New Tree Item Type
1. Add type string to `DatabaseTreeItem` switch cases in `DatabaseTreeProvider.ts`
2. Add icon mapping and children fetching logic
3. Add context menu contributions in `package.json`

---
> Source: [NexQL-OSS/core](https://github.com/NexQL-OSS/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
