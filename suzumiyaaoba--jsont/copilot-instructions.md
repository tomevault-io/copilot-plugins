## jsont

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`jsont` is a Terminal User Interface (TUI) JSON viewer built with React and Ink. The application reads JSON data from stdin or files and displays it in various interactive formats including tree view, collapsible view, and schema view with advanced query capabilities using jq and JSONata.

## Key Commands

### Development
- `npm run dev` - Run the application in development mode using tsx
- `npm run build` - Bundle the application using tsup (supports extensionless imports)
- `npm run start` - Run the compiled application from `dist/index.js`

### Testing
- `npm run test` - Run tests in watch mode
- `npm run test:run` - Run tests once and exit
- `npm run test:watch` - Run tests in explicit watch mode
- `npm run test:ui` - Open Vitest UI for interactive testing
- `npm run test:ci` - Run CI-optimized tests (memory-limited, single-threaded)
- `npm run test:coverage` - Run tests with coverage reporting (HTML, JSON, text)
- `npm run test:coverage:ui` - Open Vitest UI with coverage for interactive testing and viewing
- `npm run test -- tree` - Run specific test files matching "tree"
- `npm run test -- --coverage` - Run tests with coverage reporting (alternative syntax)

**Memory-Intensive Test Exclusions**: Several tests are excluded in CI for memory optimization:
- Performance tests, JSON processing tests, SQL converter tests, integration tests
- See `vitest.config.ts` exclude list for complete details

**Coverage Reports**: Coverage reports are generated in `./coverage/` directory:
- **Text Report**: Displayed in terminal during test execution
- **HTML Report**: Interactive web interface at `./coverage/index.html`
  - Browse file-by-file coverage with line-by-line highlighting
  - View uncovered lines, branch coverage, and function coverage
  - Color-coded visualization: red (uncovered), green (covered), orange (partial)
- **JSON Report**: Machine-readable format at `./coverage/coverage-final.json`
- Coverage metrics include: Statements, Branches, Functions, Lines

### Code Quality
- `npm run check` - Run Biome linter and formatter checks
- `npm run check:write` - Run Biome checks and apply safe fixes
- `npm run type-check` - Run TypeScript type checking only
- `npm run lint` - Lint source code only
- `npm run lint:fix` - Lint and automatically fix issues
- `npm run format` - Format source code (read-only)
- `npm run format:write` - Format and write source code

### Usage
```bash
echo '{"key": "value"}' | npm run dev
cat file.json | npm run dev
npm run dev path/to/file.json
```

## Architecture Overview

### Clean Architecture Pattern

The codebase follows a feature-driven clean architecture with clear separation of concerns:

#### Entry Point (`src/index.tsx`)
- Minimal entry point that delegates to AppService
- Uses proper error handling with `handleFatalError`

#### Core Layer (`src/core/`)
- **Services**: `AppService` orchestrates application lifecycle and initialization
- **Utils**: Comprehensive utility libraries
  - Terminal management (`terminal.ts`, `processManager.ts`)
  - Advanced stdin processing (`stdinHandler.ts`) 
  - Error handling with recovery suggestions (`errorHandler.ts`)
  - LRU caching system (`lruCache.ts`)
  - CLI argument parsing (`cliParser.ts`)
  - Data conversion system (`dataConverters/`) with SQL, XML, YAML, CSV, JSON Schema support
- **Types**: Application-wide type definitions and interfaces
- **Config**: YAML configuration system with schema validation, hot reloading, and defaults
- **Context**: React context providers for configuration management

#### Features Layer (`src/features/`)
Organized by feature domains, each containing:
- **Components**: React/Ink UI components
- **Types**: Feature-specific type definitions  
- **Utils**: Business logic and utilities
- **Tests**: Co-located test files

**Key Features:**
- `tree/` - Interactive tree view with keyboard navigation
- `collapsible/` - Collapsible JSON viewer with syntax highlighting
- `search/` - Search functionality across JSON data
- `jq/` - jq query transformation support
- `schema/` - JSON schema inference and multi-format export (JSON, YAML, CSV, XML, SQL)
- `json-rendering/` - Core JSON parsing and syntax highlighting
- `common/` - Shared components (TextInput, BaseViewer, hooks)
- `debug/` - Debug logging and viewer components
- `navigation/` - Goto navigation (gg/G) and keyboard shortcuts
- `help/` - Context-sensitive help system
- `status/` - Status bar and line number display
- `settings/` - Interactive settings TUI with live preview and validation

#### Component Architecture (`src/components/`)
- **Providers**: Application-wide context and state providers (`AppStateProvider`)
- **Content**: View routing and content management (`ContentRouter`)
- **Modals**: Modal management system (`ModalManager`)
- **Status**: Status bar and line number management (`StatusBarManager`) 
- **Keyboard**: Input handling coordination (`KeyboardManager`)

#### Hooks Architecture (`src/hooks/`)
- **State Hooks**: Application state management (`useAppState`, `useLayoutCalculations`)
- **Handler Hooks**: Keyboard input processing (`useKeyboardHandler`, `useSearchHandlers`)  
- **Calculation Hooks**: Terminal and layout calculations (`useTerminalCalculations`)
- **Export Hooks**: Data export functionality (`useExportHandlers`)

#### Store Architecture (`src/store/`)
- **Jotai Atoms**: Atomic state management for ui, search, navigation, settings, debug, jq, export
- **Typed Hooks**: Strongly-typed atom accessors for type safety
- **Provider Setup**: Jotai provider configuration and initialization

#### Data Export Capabilities
- **Supported Formats**: JSON, YAML, CSV, XML, SQL, JSON Schema
- **Data Converters**: Modular converter system in `@core/utils/dataConverters/`
- **SQL Export**: Includes schema inference, dialect support, and data transformation
- **XML Export**: Hierarchical data representation with configurable options

### State Management
- **React Hooks**: Local component state and feature-specific logic
- **Jotai Atomic State**: Global state management via atomic approach
  - `src/store/atoms/` - Atomic state definitions (ui, search, navigation, settings, debug, jq, export)
  - `src/store/hooks/` - Typed hooks for accessing atoms (useUI, useSearch, useNavigation, etc.)
  - `src/store/Provider.tsx` - Jotai provider setup
- **App-level State**: AppStateProvider in `src/components/providers/` coordinates global application state
- **State Coordination**: App.tsx and AppStateProvider orchestrate communication between features

### Keyboard Input Architecture
- **Central Dispatch**: App.tsx uses unified `useInput` hook for all keyboard input
- **Handler Delegation**: App.tsx routes input to currently active feature components
- **Handler Registration Pattern**: Features register keyboard handlers via callback props
- **Handler Architecture**: `src/hooks/handlers/` contains specialized input handlers:
  - `globalHandler.ts` - Application-wide shortcuts (quit, help, etc.)
  - `navigationHandler.ts` - Movement and navigation shortcuts
  - `searchHandler.ts` - Search mode input handling
  - `jqHandler.ts` - jq query input handling
  - `helpHandler.ts` - Help system navigation
- **Keyboard Manager**: `src/components/keyboard/KeyboardManager.tsx` coordinates handler registration
- **Advanced stdin**: Sophisticated stdin processing enables keyboard input even in pipe mode

## Technical Stack

### Core Technologies
- **Runtime**: Node.js 18+ with ES Modules
- **UI Framework**: React 19 + Ink 6.0 for terminal rendering
- **Language**: TypeScript with strictest configuration
- **Build**: tsup for bundling with path alias support
- **Testing**: Vitest with coverage reporting

### Code Quality Tools
- **Linting/Formatting**: Biome (replaces ESLint + Prettier)
  - Double quotes for JavaScript/TypeScript
  - 2-space indentation
  - Automatic import organization
  - VCS integration with Git
- **Git Hooks**: Husky + lint-staged for pre-commit quality checks
- **TypeScript**: Strict mode with path aliases (`@/*`, `@core/*`, `@features/*`, `@store/*`, `@components/*`, `@hooks/*`)

### Key Dependencies
- `neverthrow` - Result type pattern for error handling
- `node-jq` - jq query processing
- `json5` - Enhanced JSON parsing
- `es-toolkit` - Modern utility functions
- `js-yaml` - YAML configuration parsing
- `zod` - Runtime type validation
- `jotai` - Atomic state management
- `mutative` - Immutable state updates
- `defu` - Deep object merging for configuration

## Development Guidelines

### Import Standards
- **Always use extensionless imports** for TypeScript files (required by build system)
- **Path Alias Usage**: 
  - **Development**: All aliases available (`@/*`, `@core/*`, `@features/*`, `@store/*`, `@components/*`, `@hooks/*`)
  - **Build Limitation**: tsup only supports `@/*`, `@core/*`, and `@features/*` aliases
  - **Critical**: Use supported aliases to avoid build failures
- **Import Organization**: external dependencies → path aliases → relative imports
- **Example**:
  ```typescript
  import { Box, Text } from "ink";
  import type { JsonValue } from "@core/types/index";
  import { TreeView } from "@features/tree/components/TreeView";
  import { useUI } from "@store/hooks/useUI";
  import { formatData } from "./utils/formatter";
  ```

### File Organization
- Tests co-located with source files using `.spec.ts` suffix
- Feature-based organization with clear domain boundaries
- Barrel exports (`index.ts`) for clean module interfaces

### Error Handling
- Use `neverthrow` Result type pattern instead of throwing exceptions
- Safe operations return `Result<T, E>` types
- Comprehensive error context and recovery suggestions

### TypeScript Configuration
- Extends `@tsconfig/strictest` for maximum type safety
- Module resolution: "bundler" for modern import handling
- Path aliases configured in both tsconfig.json and build tools
- `any` types are intentionally used for JSON data handling

## Stdin and Keyboard Handling

The application implements sophisticated stdin processing to enable keyboard navigation in all input modes:

### Pipe Mode Navigation
- **Challenge**: When JSON is piped (`echo '...' | jsont`), stdin is consumed for data, preventing keyboard input
- **Solution**: Read-then-reinitialize strategy that restores keyboard input after JSON processing
- **Implementation**: Completely read stdin, parse JSON, then create new TTY streams for keyboard input

### Navigation Features
- **Line Navigation**: j/k, arrow keys for precise movement
- **Page Navigation**: Ctrl+f/b for half-page scrolling
- **Goto Navigation**: gg (top), G (bottom) for instant positioning
- **Feature Toggle**: T (tree view), S (schema), D (debug), comma (settings), etc.

## Performance Optimization

### Caching Strategy
- **LRU Cache**: `src/core/utils/lruCache.ts` provides efficient caching with automatic eviction
- **Schema Inference**: Cached with 200-entry LRU cache to prevent redundant processing
- **Debug Log Formatting**: Cached with 1000-entry LRU cache for improved rendering
- **React Memoization**: Strategic use of `React.memo`, `useMemo`, and `useCallback`

### Algorithm Optimizations
- **Tree Filtering**: Optimized from array methods to for-loops for large datasets
- **Search Performance**: Efficient text matching with early returns
- **Memory Management**: Controlled object creation and garbage collection

### Performance Testing
- **Comprehensive Benchmarks**: 11 performance tests covering all major operations
- **Memory Usage Monitoring**: Automated tests to prevent memory leaks
- **CI/CD Integration**: Performance regression detection in build pipeline

### CI/CD Optimization
- **Memory Constraints**: CI environment optimized for limited memory (6GB max)
- **Test Configuration**: 
  - `npm run test:ci` uses `vitest.config.ci.ts` with stricter memory limits
  - Single-threaded execution with maxThreads: 1
  - Selective test exclusions for memory-intensive tests
- **Node Options**: `NODE_OPTIONS="--max-old-space-size=6144"` for heap size control
- **Build Pipeline**: Performance regression detection with automated test exclusions

### CI Test Exclusions
The following test categories are excluded in CI for memory optimization:
```typescript
// Excluded in vitest.config.ci.ts
exclude: [
  "src/performance.spec.ts",           // Performance benchmarks
  "src/error-scenarios.spec.ts",       // Error handling scenarios  
  "src/json-processing.spec.ts",       // Large JSON processing
  "src/core/utils/stdinHandler.spec.ts", // stdin processing
  "src/library-integration.spec.ts",   // Third-party integrations
  "src/integration/**",                // Full integration tests
  "src/core/utils/dataConverters/SqlConverter.spec.ts", // SQL processing
  "src/core/services/appService.spec.ts", // App service tests
  "src/features/json-rendering/utils/syntaxHighlight.spec.ts", // Syntax highlighting
  "src/core/utils/processManager.spec.ts", // Process management
  "src/features/collapsible/utils/collapsibleJson.spec.ts", // Large data collapsing
]
```

## Configuration System

### YAML Configuration
- **Location**: `~/.config/jsont/config.yaml`
- **Hot Reloading**: Configuration changes applied without restart
- **Validation**: Zod-based schema validation with helpful error messages
- **Merging**: Deep merge with default configuration

### Configuration Structure
```yaml
display:
  interface:
    showLineNumbers: boolean
    useUnicodeTree: boolean
  json:
    indent: number
    useTabs: boolean
  tree:
    showArrayIndices: boolean
    showPrimitiveValues: boolean
    maxValueLength: number

keybindings:
  navigation:
    up: string
    down: string
    pageUp: string
    pageDown: string
```

### Interactive Settings TUI
- **Access**: Comma (`,`) key opens interactive settings editor
- **Two-pane Layout**: Settings list on left, description/help on right  
- **Live Preview**: Changes shown immediately with unsaved state tracking
- **Navigation**: Tab to switch categories, j/k to navigate fields, Enter/e to edit
- **Validation**: Real-time validation with error messages and type checking
- **Persistence**: Ctrl+S to save, Ctrl+R to reset, Ctrl+D for defaults

## Testing Strategy

- **Test Coverage**: 800+ tests across all utilities, components, and integrations
- **Test Architecture**: 
  - **Co-located Tests**: `.spec.ts` files alongside implementation
  - **Feature Testing**: Each feature has comprehensive test suites
  - **Integration Tests**: Full application testing in `src/integration/`
  - **Performance Tests**: Automated benchmarking to prevent regressions
- **Test Environment**: 
  - **Runtime**: Node.js with jsdom for DOM simulation
  - **Framework**: Vitest with globals enabled
  - **Setup**: `src/vitest.setup.ts` for test initialization
- **Memory Optimization**: 
  - **CI Configuration**: Single-threaded execution to prevent OOM errors
  - **Test Exclusions**: Memory-intensive tests excluded in CI (see vitest.config.ts)
  - **Thread Pool**: Limited to 1 thread with 30s timeout for complex tests
- **Coverage**: v8 provider with text, JSON, and HTML reporting

## Common Development Patterns

### Feature Development
1. Create feature directory with `components/`, `types/`, `utils/` structure
2. Implement core logic in utils with comprehensive tests
3. Build React components using Ink primitives
4. Add keyboard handlers and integrate with App.tsx
5. Export types and utilities via barrel files

### Adding New View Modes
1. Create feature-specific state management
2. Implement keyboard handler registration pattern
3. Add mode toggle logic to App.tsx
4. Use conditional rendering for mode switching
5. Follow existing patterns for search integration

### Adding Settings Fields
1. Add field definition to `settingsDefinitions.ts` with validation
2. Create field component in `settings/components/fields/` if needed
3. Update configuration schema in `@core/config/schema.ts`
4. Add corresponding configuration mapper in `configMapper.ts`
5. Test live preview and validation behavior

### Keyboard Handler Integration
```typescript
// In feature component
const handleKeyboardInput = useCallback((input: string, key: KeyboardInput) => {
  // Handle feature-specific keys
  return handled;
}, []);

// Register with parent
useEffect(() => {
  if (onKeyboardHandlerReady) {
    onKeyboardHandlerReady(handleKeyboardInput);
  }
}, [onKeyboardHandlerReady, handleKeyboardInput]);
```

## Future Architecture Plans

The project has plans for **UI separation architecture** to enable future GUI (Web) version support:
- **Planned Engines**: `src/core/engine/` will contain UI-agnostic business logic (JsonEngine, TreeEngine, SearchEngine)
- **Planned Adapters**: `src/core/adapters/` will provide UI abstraction layer (UIAdapter, UIController)  
- **Goal**: Enable multiple UI frontends (current TUI + future Web UI) sharing core business logic
- **Status**: Architecture planning complete, implementation pending

## Modal and UI State Management

### Modal Priority System
The application uses a layered modal system managed by `ModalManager`:
- **Highest Priority**: Debug Log Viewer (fullscreen overlay)
- **High Priority**: Settings Viewer, Export Dialogs
- **Medium Priority**: Help Viewer (conditional rendering based on other modals)
- **Lowest Priority**: Main content (hidden when high-priority modals are active)

### UI State Coordination
- **Help Visibility**: When `ui.helpVisible` is true, Property Details are automatically hidden to prevent layout conflicts
- **Modal Stacking**: Higher priority modals automatically suppress lower priority ones
- **Keyboard Input**: Only active when no blocking modals are visible

### Critical UI Patterns
```typescript
// Property Details conditional rendering pattern
{(currentMode === "tree" || currentMode === "collapsible") &&
 !ui.helpVisible && (
  <PropertyDetailsDisplay ... />
)}

// Modal priority rendering pattern  
{!debugLogViewerVisible && 
 !settingsVisible && 
 !exportDialog.isVisible && 
 children}
```

## Development Workflow Patterns

### Adding New Modal Components
1. Add modal state to appropriate Jotai atom (usually `uiAtoms`)
2. Register modal in `ModalManager` with correct priority level
3. Ensure proper keyboard input blocking in `KeyboardManager`
4. Test modal stacking behavior with existing modals

### Debugging Layout Issues
- Use Debug Mode (`D` key) to inspect terminal calculations
- Check `useTerminalCalculations` for height calculations
- Verify modal priority in `ModalManager` 
- Test with different terminal sizes using resize

### Performance Considerations for Large Files
- LRU caches are pre-configured for schema inference (200 entries) and debug logs (1000 entries)
- Virtual scrolling automatically activates for large datasets
- Use `npm run test:ci` for memory-constrained testing
- Performance tests are excluded in CI but available locally

## Important Development Reminders

- **Always use extensionless imports** - The build system requires this
- **Use supported path aliases** - `@/*`, `@core/*`, `@features/*` work in build; `@store/*`, `@components/*`, `@hooks/*` work in dev only
- **Test files use `.spec.ts` suffix** - All tests are co-located with implementation  
- **Run `npm run check` before committing** - Ensures code quality standards
- **Memory-aware testing** - Use `npm run test:ci` for memory-constrained environments
- **Never assume libraries are available** - Always check existing dependencies first
- **Use `neverthrow` Result pattern** - Avoid throwing exceptions in favor of explicit error handling
- **Follow feature-based architecture** - Keep components, types, and utils together within feature directories
- **Modal UI conflicts** - Always check if new UI elements conflict with existing modals
- **Biome formatting** - Uses double quotes, 2-space indentation, and auto-import organization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SuzumiyaAoba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
