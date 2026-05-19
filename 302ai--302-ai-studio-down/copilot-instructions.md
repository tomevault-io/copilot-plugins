## 302-ai-studio-down

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Essential Development Commands
```bash
# Start development server with hot reload
yarn dev

# Type checking (required before builds)
yarn typecheck

# Code linting and formatting (uses Biome)
yarn lint
yarn lint:fix

# Build for production
yarn build

# Install Electron dependencies (run after yarn install)
yarn install:deps

# Rebuild native modules if database issues occur
yarn electron-rebuild

# Clean development artifacts
yarn clean:dev
```

### Build Process
The build system uses a multi-step process:
1. `yarn clean:dev` - Clean previous builds
2. `yarn compile:app` - Compile with electron-vite
3. `yarn compile:packageJSON` - Generate package metadata
4. `yarn typecheck` - Validate TypeScript

Always run `yarn typecheck` before committing. The build will fail without it.

### Troubleshooting Development Issues

**Database Connection Errors (`ERR_DLOPEN_FAILED`)**:
```bash
yarn electron-rebuild  # Rebuild native modules
```

**GPU Process Errors in Headless Environments**:
```bash
ELECTRON_DISABLE_GPU=1 yarn dev  # Disable GPU acceleration
```

These errors don't affect React Compiler functionality or core development workflow.

## React Compiler Integration

This project uses React Compiler (RC) for automatic optimization of React components. The compiler is integrated via Vite's Babel plugin and runs during the build process.

### How It Works
- Components are automatically memoized where beneficial
- No manual `useMemo`/`useCallback` needed in most cases
- Compiler analyzes component dependencies and optimizes renders
- Look for ✨ badge in React DevTools to see optimized components

### Debugging Compilation Issues
If a component has unexpected behavior after compilation:
1. Add `"use no memo"` directive at the top of the component to temporarily disable optimization
2. Verify the issue is compiler-related (should disappear with directive)
3. Check for Rules of React violations - common causes of issues
4. Remove directive once underlying issue is fixed

### Development Notes
- The compiler runs automatically - no special configuration needed
- Build output will contain `react/compiler-runtime` imports when optimization occurs
- Components violating React rules are safely skipped by the compiler

## Architecture Overview

### Electron Multi-Process Architecture
- **Main Process** (`src/main/`): Node.js backend with service layer
- **Renderer Process** (`src/renderer/`): React frontend
- **Preload** (`src/preload/`): Secure IPC bridge
- **Shared** (`src/shared/`): Common types, database schema, utilities

### Service-Oriented Main Process
The main process uses dependency injection (Inversify) with 16 services:
- `ChatService` - AI conversation management
- `AttachmentService` - File upload and processing  
- `MessageService` - Message CRUD operations
- `ThreadService` - Conversation thread management
- `ModelService` - AI model provider management
- `SettingsService` - Application configuration
- `WindowService` - Desktop window management
- `TriplitService` - Database operations
- Plus 8 additional services for UI, tabs, shortcuts, toolbox, etc.

Services auto-register IPC handlers via decorators. Find service methods in `src/main/services/`.

### Database Layer (Triplit)
Real-time SQLite database with 10 collections:
- `providers` - AI service configurations (302.AI, OpenAI)
- `models` - Available AI models per provider
- `threads` - Chat conversations
- `messages` - Individual messages with streaming support
- `attachments` - File uploads with metadata
- `settings` - App configuration
- Plus collections for tabs, shortcuts, toolbox, UI state

Schema definitions in `src/shared/triplit/schemas/`. Uses versioned migrations.

### AI Provider Architecture
Pluggable provider system supporting:
- **302.AI** (primary) - Custom SDK integration
- **OpenAI** - AI SDK compatibility layer
- Provider switching at runtime
- Model discovery and caching

Find provider implementations in `src/main/api/`.

### File Processing Pipeline
Multi-format file support via adapter pattern:
- **Supported**: PDF, CSV, Excel, Word, PowerPoint, images, code files, audio
- **Processing**: Text extraction, thumbnail generation, metadata parsing
- **Storage**: SQLite BLOB storage with filename/MIME type
- **Preview**: Markdown conversion for chat integration

File processing logic in `AttachmentService` (`src/main/services/attachment-service.ts`).

## Key Development Patterns

### IPC Communication
Services expose methods to renderer via decorators:
```typescript
@ipcHandler('methodName')
async someMethod(param: string): Promise<Result> {
  // Implementation
}
```

Frontend calls via bridge:
```typescript
const result = await window.api.serviceName.someMethod(param);
```

### Data Fetching Architecture
The renderer uses a structured query system located in `src/renderer/queries/` with the following architecture:

#### Query System Structure
- **Definitions** (`src/renderer/queries/definitions/`): Pre-built query builders for each collection
- **Hooks** (`src/renderer/queries/hooks/`): React hooks for data fetching
- **Composed Hooks** (`src/renderer/queries/hooks/composed/`): Higher-level hooks that combine multiple queries
- **Core Types** (`src/renderer/queries/types.ts`): TypeScript definitions for the query system

#### Basic Query Usage
```typescript
// Use collection-specific hooks
import { useMessages, useThreads } from '@renderer/queries';

// Standard query with configuration
const { data: messages, isLoading, error } = useMessages({
  enabled: true,
  select: (data) => data.filter(m => m.threadId === threadId)
});

// Single record query
const { data: thread } = useThreadsOne(() => threadsQueries.byId(threadId));
```

#### Composed Queries
For complex data requirements, use composed hooks:
```typescript
import { useChatContext } from '@renderer/queries';

// Aggregates settings, providers, models, and UI state
const {
  selectedModel,
  selectedProvider,
  isLoading,
  isReady
} = useChatContext();
```

#### Query Features
- **Type Safety**: Full TypeScript support with collection schema types
- **Reactive Updates**: Real-time data synchronization via Triplit
- **Conditional Queries**: Enable/disable queries based on conditions
- **Data Transformation**: Built-in select transformers for data processing
- **Loading States**: Unified loading and error state management

### AI Streaming
Messages support real-time streaming via server-sent events. Check `ChatService` for streaming implementation.

### Theme System
Supports light/dark modes with system preference detection. Theme state managed in `settings` collection.

## File Structure Conventions

### Component Organization
- `src/renderer/components/business/` - Domain-specific components
- `src/renderer/components/ui/` - Reusable UI primitives
- Use React Aria Components for accessibility
- Tailwind CSS for styling

### Service Location
- Main process services: `src/main/services/[name]-service.ts`
- API integrations: `src/main/api/`
- Shared utilities: `src/shared/`

### Type Definitions
- Database types: Auto-generated from Triplit schemas
- Service types: `src/main/shared/types.ts`
- Shared types: `src/shared/types/`

## Testing and Quality

The project uses Biome for linting and TypeScript for type checking. Always run:
1. `yarn typecheck` - Validates all TypeScript
2. `yarn lint` - Checks code style and potential issues
3. `yarn lint:fix` - Auto-fixes formatting issues

## Platform-Specific Notes

### Cross-Platform Building
- Uses electron-builder for packaging
- Supports Windows (x64/ARM64), macOS (Universal/x64/ARM64), Linux (x64/ARM64)
- Build configurations in package.json scripts

### Desktop Integration
- Window state persistence via electron-window-state
- System tray integration
- File associations for supported document types
- Auto-updater integration

## Database Migrations

Triplit uses versioned schema migrations. When modifying database schema:
1. Update schema files in `src/shared/triplit/schemas/`
2. Create migration in `src/shared/triplit/migrations/`
3. Update schema version in migration manager

## Security Considerations

- AGPL-3.0 licensed (open source)
- Sandboxed renderer process
- Preload script provides limited API surface
- No direct Node.js access from renderer
- File uploads sanitized and validated

### Service-Layer Error Handling (Rust-like Result; no throw)

- Do not use `throw error` in service-layer methods.
- Always return a Rust-like Result object to indicate success/failure.
- This keeps IPC handlers stable and makes error flows explicit.

Recommended patterns:
```ts
// Without data
type ServiceResult = {
  isOk: boolean;
  errorMsg: string | null;
};

// With data
type ServiceResultWithData<T> = {
  isOk: boolean;
  errorMsg: string | null;
  data?: T;
};

// Template
try {
  // ...work...
  return { isOk: true, errorMsg: null };
} catch (error) {
  // Log the error, then return an error result
  // logger.error('ServiceName:method error', { error, ...context });
  return { isOk: false, errorMsg: extractErrorMessage(error) };
}
```

Example (aligns with `updateThreadCollected`):
```ts
async updateThreadCollected(
  _event: Electron.IpcMainEvent,
  threadId: string,
  collected: boolean,
): Promise<{ isOk: boolean; errorMsg: string | null }> {
  try {
    await this.threadDbService.updateThread(threadId, { collected });
    return { isOk: true, errorMsg: null };
  } catch (error) {
    return { isOk: false, errorMsg: extractErrorMessage(error) };
  }
}
```

## Code Quality Guidelines

### Pattern Matching with ts-pattern
Use `ts-pattern`'s `match` function to replace complex if...else... chains and switch statements for better type safety and readability.

**Bad - Multiple if...else branches:**
```ts
function handleUserAction(action: UserAction) {
  if (action.type === 'CREATE' && action.resource === 'thread') {
    return createThread(action.payload);
  } else if (action.type === 'UPDATE' && action.resource === 'thread') {
    return updateThread(action.payload);
  } else if (action.type === 'DELETE' && action.resource === 'thread') {
    return deleteThread(action.payload);
  } else if (action.type === 'CREATE' && action.resource === 'message') {
    return createMessage(action.payload);
  } else {
    throw new Error('Unknown action');
  }
}
```

**Good - Using ts-pattern:**
```ts
import { match } from 'ts-pattern';

function handleUserAction(action: UserAction) {
  return match(action)
    .with({ type: 'CREATE', resource: 'thread' }, ({ payload }) => 
      createThread(payload))
    .with({ type: 'UPDATE', resource: 'thread' }, ({ payload }) => 
      updateThread(payload))
    .with({ type: 'DELETE', resource: 'thread' }, ({ payload }) => 
      deleteThread(payload))
    .with({ type: 'CREATE', resource: 'message' }, ({ payload }) => 
      createMessage(payload))
    .exhaustive();
}
```

**Complex pattern matching example:**
```ts
import { match, P } from 'ts-pattern';

function processMessage(message: Message) {
  return match(message)
    .with({ status: 'streaming', content: P.string.minLength(1) }, (msg) => 
      renderStreamingMessage(msg))
    .with({ status: 'complete', attachments: P.array(P.string) }, (msg) => 
      renderMessageWithAttachments(msg))
    .with({ status: 'error', error: P.not(P.nullish) }, (msg) => 
      renderErrorMessage(msg))
    .otherwise((msg) => renderDefaultMessage(msg));
}
```

### Utility Functions with es-toolkit
Replace hand-written utility functions with `es-toolkit` for better performance and consistency. Use es-toolkit instead of lodash when possible.

**Array Utilities:**
```ts
// Bad - Hand-written utilities
const uniqueIds = [...new Set(items.map(item => item.id))];
const chunks = items.reduce((acc, item, i) => {
  const chunkIndex = Math.floor(i / size);
  if (!acc[chunkIndex]) acc[chunkIndex] = [];
  acc[chunkIndex].push(item);
  return acc;
}, []);

// Good - Using es-toolkit
import { uniq, chunk, groupBy } from 'es-toolkit';

const uniqueIds = uniq(items.map(item => item.id));
const chunks = chunk(items, size);
const grouped = groupBy(items, item => item.category);
```

**Object Utilities:**
```ts
// Bad - Hand-written object manipulation
const picked = Object.keys(source)
  .filter(key => keys.includes(key))
  .reduce((obj, key) => ({ ...obj, [key]: source[key] }), {});

// Good - Using es-toolkit
import { pick, omit, merge } from 'es-toolkit';

const picked = pick(source, keys);
const omitted = omit(source, unwantedKeys);
const merged = merge(target, source);
```

**Function Utilities:**
```ts
// Bad - Hand-written debounce/throttle
let timeoutId: NodeJS.Timeout;
const debouncedFn = (...args) => {
  clearTimeout(timeoutId);
  timeoutId = setTimeout(() => originalFn(...args), delay);
};

// Good - Using es-toolkit
import { debounce, throttle, once } from 'es-toolkit';

const debouncedFn = debounce(originalFn, delay);
const throttledFn = throttle(originalFn, delay);
const onceFn = once(originalFn);
```

**String Utilities:**
```ts
// Bad - Hand-written string manipulation
const camelCased = str.replace(/-([a-z])/g, (g) => g[1].toUpperCase());
const kebabCased = str.replace(/[A-Z]/g, letter => `-${letter.toLowerCase()}`);

// Good - Using es-toolkit
import { camelCase, kebabCase, snakeCase, capitalize } from 'es-toolkit';

const camelCased = camelCase(str);
const kebabCased = kebabCase(str);
const snakeCased = snakeCase(str);
const capitalized = capitalize(str);
```

**Promise Utilities:**
```ts
// Bad - Hand-written promise utilities
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

// Good - Using es-toolkit
import { delay, timeout } from 'es-toolkit';

await delay(1000);
const result = await timeout(fetchData(), 5000);
```

**Predicates and Validation:**
```ts
// Bad - Hand-written type checks
const isEmptyObject = (obj: any) => obj && typeof obj === 'object' && Object.keys(obj).length === 0;
const isNonEmpty = (arr: any[]) => Array.isArray(arr) && arr.length > 0;

// Good - Using es-toolkit
import { isEmpty, isNotEmpty, isPlainObject, isArray } from 'es-toolkit';

const empty = isEmpty(obj);
const hasItems = isNotEmpty(arr);
const isPlain = isPlainObject(obj);
const isArr = isArray(value);
```

## Git Commit Guidelines

### Commit rules when using the `@git-expert.md` agent
- Do not include any content related to Claude Code in the commit message (subject and body).
- Commit messages must contain only information directly related to the current change. Do not include tools, agents, prompts, or other meta-information.
- Commit messages do not include co-author information.

---
> Source: [302ai/302-AI-Studio_down](https://github.com/302ai/302-AI-Studio_down) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
