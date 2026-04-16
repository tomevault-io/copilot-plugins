## dtfgv6

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DJ DataForge v6 is an Excel-like data grid with a formula engine, plugin system, and multi-company support. It runs entirely client-side using TypeScript, Vite, and IndexedDB for persistence.

## Development Commands

### Core Development
```bash
npm run dev              # Start dev server (http://localhost:5173)
npm run build            # TypeScript compile + production build
npm run preview          # Preview production build (http://localhost:4173)
npm run type-check       # TypeScript type checking (no emit)
```

### Code Quality
```bash
npm run lint             # ESLint (src/**/*.ts)
npm run format           # Prettier formatting
```

### Testing
```bash
npm run test             # Run all tests (vitest)
npm run test:unit        # Unit tests only
npm run test:e2e         # Playwright E2E tests
npm run test:watch       # Watch mode
npm run coverage         # Coverage report
npm run bench            # Performance benchmarks
```

### Plugin Builds
```bash
npm run build:plugin     # Build plugin using vite.config.plugin.ts
npm run build:standalone # Build standalone version (pre + post processing)
```

### Important Notes About Builds
- **NEVER** open `dist/index.html` directly in browser (CORS policy blocks `file://` protocol)
- **ALWAYS** use `npm run preview` or a local HTTP server to test builds
- Production deployments should serve files from dist/ via HTTP server
- **Node.js version:** Requires Node.js >= 18.0.0 (specified in package.json)

## Architecture Overview

### Core Layers
```
┌─────────────────────────────────────────────────────┐
│              UI Layer (ui-manager.ts)               │
│  Grid View | Dashboard Mode | Selection | Toolbar  │
├─────────────────────────────────────────────────────┤
│                 Plugin Layer                        │
│  FX-Finance | Charts | ProLease | Custom Plugins   │
├─────────────────────────────────────────────────────┤
│          API Layer (PluginContext + EventBus)       │
│  Events | UI Integration | Storage API | Formulas  │
├─────────────────────────────────────────────────────┤
│                Core Services (Kernel)               │
│  Workbook | CalcEngine | Dashboard | Table | Grid  │
├─────────────────────────────────────────────────────┤
│        Persistence Layer (storage-utils)            │
│  IndexedDB | Companies | Workbooks | Plugin Data   │
└─────────────────────────────────────────────────────┘
```

### Kernel as Central Orchestrator

The **Kernel** is a singleton that orchestrates all system components:

```typescript
kernel.workbookManager   // Manages workbooks and sheets
kernel.calcEngine        // Formula parser and evaluator
kernel.storageManager    // IndexedDB persistence
kernel.companyManager    // Multi-company contexts
kernel.pluginHost        // Plugin loading and management
kernel.eventBus          // Pub/sub event system
kernel.sessionManager    // Session tracking
kernel.dashboardManager  // Dashboard creation and management
kernel.tableManager      // Table operations and data structures
```

Access kernel via: `import { kernel } from '@core/kernel'` or `DJDataForgeKernel.getInstance()`

### File Organization

The codebase uses **consolidated files** (not yet split into modules):

**Core Layer** (`src/@core/`):
- `types/index.ts` - All TypeScript types
- `workbook-consolidated.ts` - Workbook, Sheet, Column, Cell, WorkbookManager
- `calc-engine-consolidated.ts` - Parser, Evaluator, Registry, DAG
- `calc/parser.ts` - Tokenizer and AST builder (modular)
- `storage-utils-consolidated.ts` - Storage, Logger, Formatters, Assert
- `kernel.ts` - Kernel orchestrator, EventBus, CompanyManager
- `plugin-system-consolidated.ts` - Plugin host and interfaces
- `grid-virtual-consolidated.ts` - Virtual grid rendering
- `io-transform-consolidated.ts` - Import/export, transformations
- `dashboard-manager.ts` - Dashboard orchestration and state management
- `dashboard-widgets.ts` - Widget definitions and lifecycle
- `dashboard-renderer.ts` - Canvas/SVG rendering engine
- `widget-registry.ts` - Widget catalog and registration
- `table-manager.ts` - Table data structures and operations

**UI Layer**:
- `src/app.ts` - Application entry point and initialization
- `src/main.ts` - Vite entry point
- `src/ui-manager.ts` - UI state management and event handling

**Plugin Layer** (`src/plugins/`):
- `fx-finance-plugin.ts` - FX rates and financial formulas
- `charts-plugin.ts` - Chart.js integration
- `prolease-ifrs16-plugin.ts` - IFRS 16 lease accounting
- `chart-manager.ts` - Chart configuration manager
- `index.ts` - Plugin registry and loader

**NOTE**: Consolidated files have `// FUTURE SPLIT POINTS:` comments indicating where to split when refactoring.

### Path Aliases

TypeScript path mapping (configured in tsconfig.json and vite.config.ts):

```typescript
@core/*    → src/@core/*
@ui/*      → src/@ui/*
@plugins/* → src/@plugins/*
```

Always use path aliases when importing core modules.

## Working with the Codebase

### TypeScript Configuration

This project uses **strict mode** with aggressive linting:
- All strict flags enabled (strictNullChecks, noImplicitAny, etc.)
- No unused locals or parameters allowed
- No implicit returns
- No fallthrough cases in switch

When writing code, ensure full type safety and handle all edge cases explicitly.

### Formula System

The CalcEngine provides a full formula parser and evaluator:

**Built-in Functions** (40+ formulas across core + plugins):

**Core Functions** (20 formulas):
- Math: SOMA, MÉDIA, MÁXIMO, MÍNIMO, ARREDONDAR, ABS, RAIZ, POTÊNCIA
- Text: CONCATENAR, MAIÚSCULA, MINÚSCULA, TEXTO, NÚM.CARACT
- Logic: SE, E, OU, NÃO
- Info: ÉNÚM, ÉTEXTO, ÉVAZIO
- Count: CONT.NÚM, CONT.VALORES
- Lookup: PROCV

**Plugin Functions** (FX-Finance Plugin):
- FX Functions: FX.RATE, FX.TODAY, FX.CONVERT, FX.VARIATION, FX.AVG, FX.MAX, FX.MIN, FX.FORWARD
- Financial Functions: FIN.PV, FIN.FV, FIN.PMT, FIN.NPER, FIN.RATE, FIN.RATE.EQUIVALENT
- Lease Functions: LEASE_PV, LEASE_MONTHLY_RATE, LEASE_ROU_OPENING (ProLease plugin)

**Registering Custom Functions**:
```typescript
const registry = kernel.calcEngine.getRegistry();
registry.register('MY_FUNC', (arg1: number, arg2: number) => {
  return arg1 + arg2;
}, { argCount: 2, description: 'My custom function' });
```

**Formula Evaluation Flow**:
1. User enters: `=SOMA(A1:A10)`
2. FormulaParser tokenizes → AST
3. CalcEngine evaluates AST recursively
4. Registry lookup for SOMA
5. Arguments evaluated (A1:A10 → array)
6. Function called, result stored in cell

Formulas support dependency tracking and automatic recalculation.

### Plugin System

All plugins implement the `Plugin` interface:

```typescript
interface Plugin {
  manifest: PluginManifest;
  init(context: PluginContext): Promise<void>;
  dispose?(): Promise<void>;
}
```

Each plugin receives a `PluginContext` with access to:
- `context.kernel` - Full kernel services
- `context.storage` - Plugin-specific persistent storage (IndexedDB)
- `context.ui` - UI integration (toolbar buttons, panels, menus)
- `context.events` - Event pub/sub system

**Built-in Plugins**:
- **FX-Finance** (`src/plugins/fx-finance-plugin.ts`) - Exchange rates (PTAX/BCB API), 8 FX formulas (FX.RATE, FX.CONVERT, etc.), 6 financial formulas (FIN.PV, FIN.FV, FIN.PMT, etc.), and economic indices tracking
- **Charts Professional** (`src/plugins/charts-plugin.ts`) - Chart.js integration with 8 chart types, 5 themes, wizard-based creation, PNG export, and persistent storage
- **ProLease IFRS 16** (`src/plugins/prolease-ifrs16-plugin.ts`) - Lease accounting with IFRS 16 compliance, amortization schedules, Web Worker calculations, and contract management

### Workbook and Sheet Structure

**Workbook** → **Sheets** → **Cells**

```typescript
// Create workbook
const wb = kernel.workbookManager.createWorkbook('My Workbook');

// Add sheet
const sheet = wb.addSheet('Sheet1');

// Set cell values
sheet.setCell(0, 0, 'Name');
sheet.setCell(1, 0, 'John', { type: 'text' });

// Set formula
sheet.setCell(1, 1, '=A2&" Smith"', {
  formula: '=A2&" Smith"',
  type: 'formula'
});

// Recalculate
await kernel.recalculate(sheet.id);
```

**Storage**: Sparse data structure - empty cells are not stored. Rows and columns use `Map<number, ...>` for efficient memory usage.

### Multi-Company Support

Every workbook can belong to a company context:

```typescript
const company = await kernel.companyManager.createCompany('Acme Corp');
kernel.companyManager.setActiveCompany(company.id);

// Workbooks created now will inherit company settings
const wb = kernel.workbookManager.createWorkbook('Q4 Sales');
// wb.companyId === company.id
```

Company contexts include settings for locale, currency, date format, number format, decimals, and timezone.

### Event System

Use the EventBus for decoupled communication:

```typescript
// Listen to events
kernel.eventBus.on('workbook:saved', (data) => {
  console.log('Workbook saved:', data.workbookId);
});

// Emit events
kernel.eventBus.emit('custom:event', { data: 'payload' });
```

**Standard Events**:
- **Kernel**: `kernel:ready`, `kernel:shutdown`, `kernel:recalc-done`, `kernel:autosave-done`
- **Workbook**: `workbook:created`, `workbook:deleted`, `workbook:saved`
- **Sheet**: `sheet:created`, `sheet:deleted`, `sheet:renamed`, `sheet:activated`
- **Plugin**: `plugin:loaded`, `plugin:unloaded`
- **Cell**: `cell:changed`, `cell:edited`
- **Dashboard**: `dashboard:created`, `widget:added`, `widget:updated`

### IndexedDB Persistence

All data persists to IndexedDB (`DJ_DataForge_v6` database):

**Object Stores**:
- `companies` - Company contexts
- `workbooks` - Workbook data (indexed by companyId)
- `snapshots` - Recovery snapshots
- `plugin_data` - Plugin-specific data (indexed by pluginId)
- `plugin_settings` - Plugin configurations
- `settings` - Global settings

**Auto-save**: Runs every 10 seconds automatically. Use `kernel.saveWorkbook(id)` or `kernel.saveAllWorkbooks()` for manual saves.

### Dashboard System

DJ DataForge v6 includes a dashboard system for creating interactive data visualizations and KPI displays:

**Dashboard Manager** (`kernel.dashboardManager`):
- Create and manage dashboards within workbooks
- Widget lifecycle management (add, remove, update)
- State persistence to IndexedDB
- Dashboard-to-sheet rendering

**Widget System**:
- **Widget Registry**: Central catalog of available widget types
- **Widget Definitions**: Reusable widget templates with configuration schemas
- **Widget Renderer**: Canvas/SVG rendering engine for visualizations
- **Widget Types**: KPI cards, charts, tables, gauges, sparklines, and custom widgets

**Usage Example**:
```typescript
// Create dashboard
const dashboard = kernel.dashboardManager.createDashboard('Sales Dashboard');

// Add KPI widget
dashboard.addWidget({
  type: 'kpi-card',
  config: {
    title: 'Total Revenue',
    dataSource: '=SOMA(Sales!D:D)',
    format: 'currency'
  }
});

// Render to sheet
await dashboard.renderToSheet('Dashboard Sheet');
```

**Plugin Integration**: Dashboard widgets can be registered by plugins, allowing custom visualizations beyond the built-in types.

### UI Manager

The UI Manager (`src/ui-manager.ts`) handles all UI state and user interactions:

**Key Responsibilities**:
- **Mode Management**: Switch between Grid, Dashboard, and Plugin views
- **Selection State**: Track active cells, ranges, and sheets
- **Toolbar State**: Manage toolbar buttons, menus, and actions
- **Panel Management**: Side panels, dialogs, and modals
- **Keyboard Shortcuts**: Excel-like keyboard navigation (Arrow keys, F2, Ctrl+C/V, etc.)
- **Ribbon Controls**: Context-sensitive toolbars based on selection

**Dashboard Mode**:
- Toggle with ribbon button or programmatically
- Hides grid, shows dashboard canvas
- Enables widget interaction and editing
- Persists mode preference per workbook

**Integration with Kernel**:
```typescript
// Access UI state
const uiManager = window.DJUIManager;

// Switch to dashboard mode
uiManager.setMode('dashboard');

// Get current selection
const selection = uiManager.getSelection();

// Register custom keyboard shortcut
uiManager.registerShortcut('Ctrl+Shift+D', () => {
  // Custom action
});
```

## Common Tasks

### Adding a New Formula

1. Open `src/@core/calc-engine-consolidated.ts`
2. Find the `FormulaRegistry` constructor
3. Add registration:
```typescript
this.register('MY_FORMULA', (arg1: number, arg2: number) => {
  return arg1 * arg2;
}, { argCount: 2, description: 'Multiplies two numbers' });
```

### Creating a New Plugin

1. Create `src/plugins/my-plugin.ts`:
```typescript
import type { Plugin, PluginContext, PluginManifest } from '@core/types';

export class MyPlugin implements Plugin {
  manifest: PluginManifest = {
    id: 'my-plugin',
    name: 'My Plugin',
    version: '1.0.0',
    author: 'Your Name',
    description: 'Does something cool',
    permissions: ['read:workbook', 'write:workbook', 'ui:toolbar'],
    entryPoint: 'my-plugin.js',
  };

  async init(context: PluginContext): Promise<void> {
    // Register custom formulas
    const registry = context.kernel.calcEngine.getRegistry();
    registry.register('MY_FUNC', (arg: number) => arg * 2, {
      argCount: 1,
      description: 'Doubles a number'
    });

    // Add UI elements
    context.ui.addToolbarButton({
      id: 'my-button',
      label: 'My Action',
      icon: '🚀',
      onClick: () => this.handleClick(context)
    });

    context.ui.showToast('My Plugin loaded!', 'success');
  }

  private handleClick(context: PluginContext): void {
    // Plugin logic here
  }

  async dispose(): Promise<void> {
    // Cleanup resources
  }
}
```

2. Register in `src/plugins/index.ts`:
```typescript
import { MyPlugin } from './my-plugin';
export const myPlugin = new MyPlugin();
```

3. Load in `src/app.ts` during kernel initialization:
```typescript
await kernel.pluginHost.loadPlugin(myPlugin);
```

### Working with Sheets Programmatically

```typescript
// Get active workbook
const wb = kernel.workbookManager.getActiveWorkbook();
if (!wb) return;

// Add sheet with data
const sheet = wb.addSheet('Data');
const headers = ['ID', 'Name', 'Amount'];
headers.forEach((h, col) => sheet.setCell(0, col, h));

// Add data rows
const data = [[1, 'Alice', 100], [2, 'Bob', 200]];
data.forEach((row, rowIdx) => {
  row.forEach((val, colIdx) => {
    sheet.setCell(rowIdx + 1, colIdx, val);
  });
});

// Recalculate and save
await kernel.recalculate(sheet.id);
await kernel.saveWorkbook(wb.id);
```

### Using FX-Finance Plugin Features

The FX-Finance plugin provides exchange rate formulas and financial calculations:

```typescript
// Get exchange rate for a specific date
sheet.setCell(0, 0, '=FX.RATE("USD", "2025-01-15")');

// Get today's rate
sheet.setCell(1, 0, '=FX.TODAY("EUR")');

// Convert currency amounts
sheet.setCell(2, 0, '=FX.CONVERT(1000, "USD", "BRL", "2025-01-15")');

// Calculate variation between dates
sheet.setCell(3, 0, '=FX.VARIATION("USD", "2025-01-01", "2025-01-15")');

// Financial formulas
sheet.setCell(4, 0, '=FIN.PV(0.05/12, 60, -500)'); // Present Value
sheet.setCell(5, 0, '=FIN.FV(0.05/12, 60, -500)'); // Future Value
sheet.setCell(6, 0, '=FIN.PMT(0.05/12, 60, 10000)'); // Payment

// Sync PTAX rates from BCB (Brazilian Central Bank)
// This is done via UI button or programmatically:
const fxPlugin = kernel.pluginHost.getPlugin('fx-finance');
await fxPlugin.syncPTAX('2025-01-01', '2025-12-31');
```

**FX Rate Cache**: Rates are cached in IndexedDB for fast O(1) lookups. Manual rates can be added via the plugin UI for currencies not covered by PTAX.

### Testing Changes

1. Run `npm run type-check` first to catch TypeScript errors
2. Use `npm run dev` to test in browser
3. Check browser console for errors (Logger outputs there)
4. Use `npm run test` for automated tests
5. Use `npm run preview` to test production build

## Debugging and Logging

The project includes a Logger utility:

```typescript
import { logger } from '@core/storage-utils-consolidated';

logger.info('Message', { data: 'optional object' });
logger.warn('Warning');
logger.error('Error occurred', error);
logger.debug('Debug info');

// Get log history
const logs = logger.getHistory({ level: 'error' });

// Set log level
logger.setLevel('debug'); // 'debug' | 'info' | 'warn' | 'error'
```

All logs appear in browser console and are stored in memory (can be exported).

## Development Guidelines

### File Size Limits
- Keep TypeScript files under **500 lines** where possible
- Consolidated files are temporary; split when appropriate
- Use `// FUTURE SPLIT POINTS:` comments to mark logical boundaries

### Code Style
- Use TypeScript strict mode (no `any` without good reason)
- Prefer explicit types over inference for public APIs
- Use functional programming patterns where appropriate
- Comment complex logic, especially in formula evaluation

### Naming Conventions
- Classes: PascalCase (`WorkbookManager`)
- Functions/methods: camelCase (`createWorkbook`)
- Constants: UPPER_SNAKE_CASE (`MAX_CELL_SIZE`)
- Private fields: prefix with `_` or use `private` keyword
- Files: kebab-case (`calc-engine-consolidated.ts`)

### Testing
- Unit tests for core logic (calc engine, workbook operations)
- E2E tests for user workflows (Playwright)
- Performance benchmarks for critical paths (virtual grid, formula evaluation)

## Project Roadmap

The project is currently in **Phase 1 (Foundation)** with these remaining phases:

- **Phase 2**: Advanced calc engine (financial functions, date functions)
- **Phase 3**: I/O & Transform (CSV/XLSX/JSON import/export, filters, undo/redo)
- **Phase 4**: UI Shell (virtual grid with canvas, cell editor, copy/paste)
- **Phase 5**: Plugin system enhancements (plugin marketplace, security)
- **Phase 6**: Polish & QA (performance tuning, accessibility, documentation)

## Additional Documentation

Comprehensive specifications are available in the project root:
- SPEC-00 to SPEC-11 (architecture, API, UI/UX, plugin system, testing)
- ARCHITECTURE_ANALYSIS.md - Complete architectural overview
- PROLEASE_V6_README.md - ProLease plugin documentation
- PROLEASE_IMPLEMENTATION_GUIDE.md - IFRS 16 implementation details

When implementing new features, consult the relevant SPEC documents for detailed requirements and design decisions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/syorankun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
