## sales-demo-glowco

> Component organization patterns - when to break out into folders, where to put constants and types


# Frontend Component Organization Rules

## When to Break Out into a Folder

Break a component into its own folder when **any** of these conditions are met:

1. **Multiple related components** - Component has 3+ child components or sub-components
2. **Shared constants/types** - Component has constants or types that are used by multiple child components
3. **Complex component** - Component file exceeds 300-400 lines and has clear logical sections
4. **Reusable sub-components** - Component has sub-components that might be reused elsewhere
5. **Multiple related files** - Component needs utilities, hooks, or helpers specific to it

### Examples

✅ **BREAK OUT** - DashboardLayout has multiple components (NavItem, PageHeader, Sidebar, etc.)
```
layouts/
  DashboardLayout/
    DashboardLayout.tsx    # Main component
    NavItem.tsx           # Sub-component
    PageHeader.tsx         # Sub-component
    Sidebar.tsx            # Sub-component
    PageContent.tsx        # Sub-component
    HeaderWithOrderDate.tsx # Sub-component
    constants.ts           # Shared constants
    types.ts              # Shared types
    index.ts              # Public exports
```

✅ **BREAK OUT** - BuyerOps container has multiple tables, nested sub-folders, and extracted hooks
```
containers/
  BuyerOps/
    index.tsx                   # Main container entry point
    BuyerOpsContent.tsx         # Orchestration component
    BuyerOpsDataLoader.tsx      # Data loading wrapper
    constants.ts                # Shared constants (transform IDs, filter text)
    types.ts                    # Shared types (BuyerOpsParams, etc.)
    utils.ts                    # Shared utility functions
    hooks/                      # ← Hooks subdirectory for BuyerOps-level hooks
      index.ts                  # Re-exports all hooks
      useBuyerOpsSave.ts        # Save/cancel/edit-state management
    FilterBar/                  # ← Nested component folder (filter controls)
      index.ts                  # Barrel — public API (exports FilterBar only)
      FilterBar.tsx             # Main filter bar component
      FilterSelect.tsx          # FilterSelect + StatusFilterSelect sub-components
      utils.ts                  # Filter handler helpers (getFilterHandler, etc.)
    ProductTable/               # ← Nested component folder
      index.ts                  # Barrel — public API for external consumers
      ProductTable.tsx          # Main table component (composes sub-components)
      ProductGroupRow.tsx       # Row renderer for group rows
      ProductDetailRow.tsx      # Row renderer for detail/expansion rows
      SummaryTable.tsx          # PO-level summary table
      EditAllDialog.tsx         # Bulk-edit dialog
      BatchesEditableCell.tsx   # Editable cell for batches
      EditableCell.tsx          # Generic editable cell
      constants.ts              # Column definitions, numeric columns
      types.ts                  # ItemData, FlattenedRow, SummaryData
      utils.ts                  # Key-generation helpers
      hooks/                    # ← Hooks grouped in subdirectory
        index.ts                # Re-exports all hooks
        useBuyerOpsEdits.ts     # Edit state machine
        useEditTracking.ts      # Per-row edit counters + global count
        useScrollPreservation.ts # Virtuoso scroll + expand/collapse restoration
        useSummaryData.ts       # Summary computation
      SupplyWalkTable/          # ← Nested sub-component folder
        index.ts                # Public exports (component + cache + types)
        SupplyWalkTable.tsx
        SupplyWalkTableHeader.tsx
        SupplyWalkTableRow.tsx
        ExpirationWarningBanner.tsx
        supplyWalkCache.ts      # Cache singleton lives with its feature
        constants.ts            # supplyWalkTheme
        types.ts                # SupplyWalkCacheEntry, SupplyWalkCacheRef, SupplyWalkTableProps
        utils.ts                # formatValue, getCellStyle, buildRecalculationInfo, sortRowsByMeasure
        hooks/                  # ← Hooks grouped in subdirectory
          index.ts              # Re-exports all hooks
          useSupplyWalkCache.ts       # Cache management for Virtuoso remounts
          useSupplyWalkRecalculation.ts # Recalculation when CORRECTED_BATCHES changes
    SupplierTable/              # ← Nested component folder
      index.ts                  # Barrel — public API
      SuppliersTable.tsx        # Main table component
      SupplierTableRow.tsx      # Row renderer with diff formatting
      constants.ts              # COLUMN_ORDER, NUMERIC_COLUMNS
      types.ts                  # SuppliersTableProps, SupplierRow
      hooks/                    # ← Hooks always in subdirectory
        index.ts                # Re-exports all hooks
        useScrollPreservation.ts # Virtuoso scroll position preservation
    POTable/                    # ← Nested component folder
      index.ts                  # Barrel — public API
      POTable.tsx               # Main table component
      TableRow.tsx              # Row renderer
      constants.ts              # COLUMN_ORDER, STATUS_CONFIG
      types.ts                  # POTableProps, SupplierLocationRow, WarningFlags
      utils.ts                  # Warning logic, min-value helpers
      hooks/                    # ← Hooks always in subdirectory
        index.ts                # Re-exports all hooks
        useRowSelection.ts      # Row selection state + select all
        useScrollPreservation.ts # Virtuoso scroll position preservation
```

❌ **DON'T BREAK OUT** - Simple component with no sub-components
```
components/
  Logo.tsx                # Single file is fine
  Button.tsx              # Single file is fine
```

## File Organization Within Component Folders

### Standard Structure

```
ComponentName/
  index.ts               # Barrel — public API (external consumers import from here)
  ComponentName.tsx       # Main component
  SubComponent1.tsx      # Child components
  SubComponent2.tsx
  constants.ts           # Constants and configuration
  types.ts               # TypeScript types and interfaces
  utils.ts               # Utility functions (if needed)
  hooks/                 # Custom hooks — always in a hooks/ subdirectory
    index.ts             # Re-exports all hooks
    useFeatureA.ts
    useFeatureB.ts
```

### File Placement Rules

1. **Main Component**
   - Named `ComponentName.tsx` or `index.tsx`
   - Contains the primary component logic
   - Should be the default export

2. **Constants File** (`constants.ts`)
   - **Location**: Same level as main component
   - **Contains**:
     - Configuration objects (e.g., `PAGE_CONFIG`, `NAVIGATION_ITEMS`)
     - Default values
     - Magic numbers/strings
     - Helper functions that return constants (e.g., `formatLastUpdated`)
   - **When to create**: When constants are used by multiple components in the folder

3. **Types File** (`types.ts`)
   - **Location**: Same level as main component
   - **Contains**:
     - TypeScript interfaces
     - Type aliases
     - Props types for components in the folder
   - **When to create**: When types are shared by multiple components in the folder

4. **Index File** (`index.ts`)
   - **Location**: Same level as main component
   - **Purpose**: Public API for the component folder
   - **Exports**: Main component (default) and any public sub-components/types

5. **Custom Hooks** (`hooks/use*.ts`)
   - **One hook per file**, named `useFeatureName.ts` (e.g., `useEditTracking.ts`, `useSummaryData.ts`)
   - **Always place hooks in a `hooks/` subdirectory** with its own `index.ts` barrel — even if there is only one hook
   - Hooks should accept explicit parameters (not reach into parent state) to stay testable

6. **Module-Level Singletons / Caches** (e.g., `supplyWalkCache.ts`)
   - **Must live in their own file** — never in a component file (see Fast Refresh rule below)
   - **Place with the feature they belong to** (e.g., supply walk cache lives in `SupplyWalkTable/`, not the parent folder)
   - Contains refs, caches, or factory functions that persist across component remounts
   - Export the singleton and a clear/reset function; re-export via the folder's `index.ts` barrel

### Fast Refresh Compatibility

**CRITICAL**: Vite's Fast Refresh (React Refresh) only works when a file exports **only React components**.

- ❌ Exporting a component **and** a module-level ref/cache/function from the same `.tsx` file will disable Fast Refresh for that file.
- ✅ Move non-component exports (caches, singletons, utility functions, type re-exports) to a separate `.ts` file.

```typescript
// ❌ BAD - ProductTable.tsx exports component + cache
export const supplyWalkCacheRef = { current: {} };
export const clearSupplyWalkCache = () => { ... };
export const ProductTable: FC = () => { ... };

// ✅ GOOD - cache in its own file, co-located with its feature
// SupplyWalkTable/supplyWalkCache.ts
export const supplyWalkCacheRef = { current: {} };
export const clearSupplyWalkCache = () => { ... };

// SupplyWalkTable/index.ts - re-exports cache alongside component
export { SupplyWalkTable } from "./SupplyWalkTable";
export { supplyWalkCacheRef, clearSupplyWalkCache } from "./supplyWalkCache";

// ProductTable.tsx - imports cache from the SupplyWalkTable barrel
import { supplyWalkCacheRef } from "./SupplyWalkTable";
export const ProductTable: FC = () => { ... };
```

### Nested Component Folders

When a sub-component within a folder grows complex enough to warrant its own folder, nest it:

```
ProductTable/
  index.ts                     # Barrel for external consumers
  ProductTable.tsx             # Main component
  hooks/                       # Hooks subdirectory (3+ hooks)
    index.ts
    useBuyerOpsEdits.ts
    useEditTracking.ts
    useSummaryData.ts
  SupplyWalkTable/             # Nested sub-component folder
    index.ts                   # Re-exports component + cache + types
    SupplyWalkTable.tsx
    SupplyWalkTableRow.tsx
    supplyWalkCache.ts         # Cache co-located with its feature
    constants.ts
    types.ts
    utils.ts
    hooks/                     # Hooks subdirectory (2+ hooks in a nested folder)
      index.ts
      useSupplyWalkCache.ts
      useSupplyWalkRecalculation.ts
```

- Keep nesting to **at most 2 levels** deep (e.g., `BuyerOps/ProductTable/SupplyWalkTable/`)
- Every folder must have an `index.ts` barrel — parent components import from the folder, not individual files
- Singletons and caches live with the feature they belong to, not in the parent folder

### Constants vs Types Placement

#### Constants (`constants.ts`)
- Configuration objects
- Default values
- Magic numbers/strings
- Helper functions that return constants
- Static data arrays/objects

```typescript
// ✅ GOOD - constants.ts
export const NAVIGATION_ITEMS: NavigationItem[] = [...]
export const DEFAULT_PAGE_TITLE = "Buyer Operations"
export const formatLastUpdated = (date: string): string => {...}
```

#### Types (`types.ts`)
- TypeScript interfaces
- Type aliases
- Props types
- Shared type definitions

```typescript
// ✅ GOOD - types.ts
export interface NavigationItem {
  id: string
  icon: ComponentType<any>
  label: string
  href: string
}

export interface PageHeaderProps {
  pageTitle: string
  pageDescription: string
}
```

## Import Patterns

### Within Component Folder
- Import from relative paths: `./constants`, `./types`, `./SubComponent`
- Import hooks from the hooks barrel: `import { useEditTracking, useSummaryData } from "./hooks"`
- Import from nested folders via their barrel: `import { supplyWalkCacheRef } from "./SupplyWalkTable"`

### From Outside Component Folder
- **Always import from the barrel** (`index.ts`): `import { ProductTable, clearSupplyWalkCache } from "./ProductTable"`
- ❌ Never reach into internal files: `import { ProductTable } from "./ProductTable/ProductTable"`

## Examples

### ✅ GOOD - Organized Component Folder

```
layouts/
  DashboardLayout/
    DashboardLayout.tsx    # Main component
    NavItem.tsx            # Sub-component
    PageHeader.tsx         # Sub-component
    Sidebar.tsx            # Sub-component
    constants.ts           # NAVIGATION_ITEMS, PAGE_CONFIG, etc.
    types.ts               # NavigationItem, PageHeaderProps, etc.
    index.ts               # export { default } from './DashboardLayout'
```

### ✅ GOOD - Simple Component (No Folder Needed)

```
components/
  Logo.tsx                 # Single file, no folder needed
  Button.tsx              # Single file, no folder needed
```

### ❌ BAD - Constants/Types in Wrong Place

```
layouts/
  DashboardLayout.tsx
  dashboardLayoutConstants.ts  # ❌ Should be in folder
  dashboardLayoutTypes.ts     # ❌ Should be in folder
```

### ❌ BAD - Constants/Types in Component File

```typescript
// ❌ Don't put constants in component file
const NAVIGATION_ITEMS = [...]  // Should be in constants.ts
interface NavItemProps {...}     // Should be in types.ts
```

## Migration Checklist

When refactoring a component:

- [ ] Component has 3+ sub-components or exceeds 300 lines? → Create folder
- [ ] Created folder? → Add `index.ts` barrel for external consumers
- [ ] Constants used by multiple components? → Create `constants.ts`
- [ ] Types shared by multiple components? → Create `types.ts`
- [ ] Any custom hooks? → Always create `hooks/` subdirectory with `index.ts` barrel
- [ ] Module-level singletons or caches? → Separate `.ts` file co-located with the feature it serves
- [ ] Component file only exports React components? → If not, extract non-component exports (Fast Refresh)
- [ ] Sub-component complex enough for its own folder? → Nest, add `index.ts` barrel
- [ ] External consumers import from barrel (`./ProductTable`), not internal files?
- [ ] Updated all imports in consuming files?
- [ ] Removed stale/old files after renames or moves?

## Reference Examples

- **Complex Layout**: [DashboardLayout](mdc:frontend/src/layouts/DashboardLayout/)
- **Container with nested table folders**: [BuyerOps](mdc:frontend/src/containers/BuyerOps/)
- **Decomposed table with hooks and cache**: [ProductTable](mdc:frontend/src/containers/BuyerOps/ProductTable/)
- **Nested sub-component folder with barrel**: [SupplyWalkTable](mdc:frontend/src/containers/BuyerOps/ProductTable/SupplyWalkTable/)
- **Table with hooks subdirectory**: [POTable](mdc:frontend/src/containers/BuyerOps/POTable/)
- **Filter bar with sub-components**: [FilterBar](mdc:frontend/src/containers/BuyerOps/FilterBar/)
- **UI component with styles/types/hooks split**: [searchable-filter-select](mdc:frontend/src/components/ui/searchable-filter-select/)
- **Simple Component**: `frontend/src/components/Logo.tsx`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atomic-supply) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
