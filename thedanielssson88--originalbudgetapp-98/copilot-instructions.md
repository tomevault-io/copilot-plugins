## originalbudgetapp-98

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
npm run dev       # Start development server with tsx (port 5000)
npm run build     # Build for production (Vite frontend + esbuild backend)
npm run start     # Start production server from dist/
npm run check     # TypeScript type checking
npm run db:push   # Push Drizzle schema changes to PostgreSQL
```

## Architecture Overview

Full-stack Swedish budget management application with PostgreSQL backend and React frontend.

### Tech Stack
- **Database**: PostgreSQL with Drizzle ORM (Neon serverless), schema in `shared/schema.ts`
- **Backend**: Express.js + TypeScript, mock auth (`dev-user-123`)
- **Frontend**: React + TypeScript + TanStack Query + shadcn/ui + Wouter routing
- **Build**: Vite (frontend) + esbuild (backend)
- **State**: Orchestrator pattern + TanStack Query for server state

### Key Directories
- `server/` - Express backend (`index.ts`, `routes.ts`, `dbStorage.ts`)
- `client/src/` - React frontend application
- `shared/` - Drizzle schemas and Zod validators
- `client/src/orchestrator/` - Core state management logic
- `client/src/hooks/` - TanStack Query hooks for API operations

## Database Architecture

UUID-based entities with Drizzle ORM. Critical tables:
- `users`, `familyMembers` - Multi-user household support
- `accounts` - Bank accounts with CSV import mappings
- `transactions` - Central transaction storage with UUID categories
- `huvudkategorier`, `underkategorier` - Swedish category hierarchy
- `categoryRules` - Auto-categorization engine
- `monthlyBudgets`, `budgetPosts` - Monthly budget configurations
- `monthlyAccountBalances` - Payday-calculated balances (25th monthly)
- `bankCsvMappings` - Persistent column mappings for bank imports

## Core Systems

### State Management Architecture
- **Orchestrator Pattern**: `client/src/orchestrator/budgetOrchestrator.ts` - Central state mutations
- **Budget State**: `client/src/state/budgetState.ts` - Single source of truth with `allTransactions`
- **TanStack Query**: Server state with React hooks in `client/src/hooks/`
- **API Store**: `client/src/store/apiStore.ts` - API operation coordination

### Transaction Import System
1. **File Processing**: CSV/XLSX via `TransactionImportEnhanced.tsx`
2. **Column Mapping**: Stored in `bankCsvMappings` table per bank
3. **Smart Reconciliation**: Duplicate detection and merge logic
4. **Balance Calculation**: Swedish payday logic (25th of month)
5. **Auto-Categorization**: UUID-based rules via `categoryRules`
6. **Persistence**: PostgreSQL with full audit trail

### Category Management
- **UUID-Based**: Eliminates naming conflicts, full migration from legacy strings
- **Hierarchical**: `huvudkategorier` → `underkategorier` structure  
- **Migration Service**: `categoryMigrationService.ts` handles legacy upgrades
- **CRUD Operations**: Full management via `/api/huvudkategorier` routes

## API Routes (`server/routes.ts`)

Mock auth middleware injects `dev-user-123` for all routes:
- **Bootstrap**: `/api/bootstrap` - Initial app data load
- **Categories**: `/api/huvudkategorier`, `/api/underkategorier` 
- **Accounts**: `/api/accounts` - Bank account management
- **Transactions**: `/api/transactions`, `/api/transactions/synchronize`
- **Rules**: `/api/category-rules` - Auto-categorization rules
- **Budgets**: `/api/monthly-budgets`, `/api/budget-posts`
- **Balances**: `/api/monthly-account-balances` - Payday balance tracking
- **Import**: `/api/banks`, `/api/bank-csv-mappings`

## Environment Setup

```bash
DATABASE_URL  # PostgreSQL connection (required for production)
PORT         # Server port (default: 5000)
NODE_ENV     # development/production
```

## Critical Implementation Details

### UUID Migration (2025)
- Complete migration from string-based to UUID identifiers
- `CategoryMigrationDialog.tsx` guides users through upgrade
- Backward compatibility during transition period

### Swedish Payday Logic
- Monthly balances calculated on 25th (Swedish payday standard)
- `monthlyAccountBalances` table stores calculated vs actual vs bank balances
- Month keys format: `YYYY-MM`

### Intelligent Import System
- Multi-format support (CSV/XLSX) with encoding detection
- Bank-specific column mappings with persistent storage
- Reconciliation prevents duplicate transactions
- Real-time balance calculation and verification

## Adding New Features

### Database Changes
1. Add table schema in `shared/schema.ts` with UUID primary key
2. Create Zod insert/select schemas with proper validation
3. Implement CRUD operations in `server/dbStorage.ts`
4. Add API routes in `server/routes.ts` with mock auth
5. Create TanStack Query hook in `client/src/hooks/`

### Frontend Pages
1. Create component in `client/src/pages/`
2. Add route to `client/src/App.tsx` Switch component
3. Add navigation item in `client/src/components/AppSidebar.tsx`

### Transaction Processing
- **Core Logic**: `budgetOrchestrator.ts` - All state mutations
- **Import UI**: `TransactionImportEnhanced.tsx` - File upload and mapping
- **Calculations**: `client/src/services/calculationService.ts` - Balance logic

## Field Mapping Management (CRITICAL)

### MOST COMMON BUG: New fields not displaying in UI
When adding new fields to ANY entity (transactions, accounts, etc.), the data flow is complex and requires updates in MULTIPLE places. Missing any step will cause fields to disappear or show as empty/null.

**ROOT CAUSE**: Manual object mapping without including all fields from the API/database.

### UNIVERSAL CHECKLIST for New Fields

#### 1. Backend Data Retrieval (`server/dbStorage.ts`)
**CRITICAL**: Never use explicit column selection in bulk queries with Drizzle ORM
```typescript
// ❌ WRONG - Fields will be missing/null
const result = await db.select({
  id: transactions.id,
  newField: transactions.newField,
}).from(transactions)

// ✅ CORRECT - All fields included
const result = await db.select().from(transactions)
```

#### 2. Frontend Data Mapping - SEARCH ALL .map() FUNCTIONS
**CRITICAL**: Search codebase for ALL `.map(item => ({` patterns and add new fields to EVERY conversion.

**EXAMPLE BUG (January 2025)**: `TransactionImportEnhanced.tsx:718` mapped accounts but omitted `lastUpdate` field:
```typescript
// ❌ WRONG - Missing lastUpdate field
const accounts: Account[] = accountsFromAPI.map(acc => ({
  id: acc.id,
  name: acc.name,
  // Missing: lastUpdate: acc.lastUpdate
}));

// ✅ CORRECT - Include ALL fields
const accounts: Account[] = accountsFromAPI.map(acc => ({
  id: acc.id,
  name: acc.name,
  lastUpdate: acc.lastUpdate, // MUST include new fields
}));
```

#### 3. Transaction Processing Pipeline (`budgetOrchestrator.ts`)
**CRITICAL**: Search for ALL `.map(tx => ({` patterns and add new fields to every conversion:

- `forceReloadTransactions()` around line 1960
- `setTransactionsForMonth()` around line 3260  
- Any other ImportedTransaction → Transaction conversions

```typescript
// Must add new field in ALL conversion points
const transactionsAsBaseType: Transaction[] = transactions.map(tx => ({
  id: tx.id,
  linkedTransactionId: tx.linkedTransactionId,
  linkedCostId: tx.linkedCostId,           // Example existing field
  savingsTargetId: tx.savingsTargetId,     // Example existing field 
  newField: tx.newField,                   // NEW FIELD - Add everywhere
  correctedAmount: tx.correctedAmount,
}))
```

#### 3. Transaction Linking Functions (`budgetOrchestrator.ts`)
**CRITICAL**: Update ALL linking functions to set new fields:
- `linkExpenseAndCoverage()`
- `coverCost()`
- `applyExpenseClaim()`
- Any other functions that create transaction relationships

```typescript
// Example: linkExpenseAndCoverage function
updates: {
  type: 'ExpenseClaim',
  correctedAmount: newNegativeCorrectedAmount,
  linkedTransactionId: positiveTxId,
  linkedCostId: positiveTxId,         // Existing relationship field
  newField: newValue,                 // NEW FIELD - Add to updates
  isManuallyChanged: true
}
```

#### 4. Frontend Snake_case Conversion (`TransactionExpandableCard.tsx`)
**CRITICAL**: Add snake_case → camelCase conversion for database compatibility
```typescript
// In useEffect conversion logic
let newField = propTransaction.newField || (propTransaction as any).new_field || null;

const convertedTransaction = {
  ...propTransaction,
  newField: newField,  // Add converted field
}
```

### UNIVERSAL Checklist for New Fields (Any Entity)

- [ ] Add field to database schema (`shared/schema.ts`)
- [ ] Verify backend uses `select()` not `select({specific})` in `dbStorage.ts`  
- [ ] **CRITICAL**: Search ENTIRE codebase for `.map(item => ({` and add field to ALL conversions
- [ ] Search for entity-specific processing functions and add field everywhere
- [ ] Add snake_case → camelCase conversion if needed
- [ ] Test both new records AND existing records work
- [ ] If old records have null values, consider adding a frontend workaround

### Why This Bug Keeps Happening
1. **Multiple data conversion points**: Data passes through DB → API → Components → UI
2. **Manual object mapping**: Developers manually reconstruct objects instead of spreading all fields
3. **No TypeScript enforcement**: Missing fields don't cause compile errors, just runtime nulls
4. **Forgotten conversions**: Easy to miss one of many `.map()` functions in large codebase

### Future Prevention Strategy
**ALWAYS search for `.map(` when adding new fields to ANY entity. This is the #1 source of "field works in API but not in UI" bugs.**

Example search commands:
```bash
# Find ALL object mapping functions
grep -rn "\.map.*=> ({" client/src/
grep -rn "\.map.*=>" client/src/ | grep "{"
```
4. **Snake_case vs camelCase**: Database uses snake_case, frontend uses camelCase

### Example: linkedCostId Fix (January 2025)
This exact issue occurred with `linkedCostId` and `correctedAmount` fields for ExpenseClaim/CostCoverage transactions. Required fixes at:
1. `server/dbStorage.ts` - Switch to `select()` from explicit columns
2. `budgetOrchestrator.ts:3274` - Add `linkedCostId` to transaction conversion
3. `budgetOrchestrator.ts:3209` - Add `linkedCostId` to linking updates  
4. `TransactionExpandableCard.tsx:80` - Extend workaround for ExpenseClaim

### Case Study: linkedPerson Field Fix (August 2025)
**Problem**: Payment transactions could be linked to family members via "Koppla Utbetalning" feature, but the linkedPerson field would not persist after page refresh. The field was saved to database correctly but disappeared from the UI.

**Root Cause**: Classic field mapping issue - linkedPerson was missing from multiple frontend data conversion functions, causing it to be filtered out during data processing.

**Symptoms**:
- ✅ Backend API correctly saved linkedPerson to `linked_person` database column
- ✅ Direct API calls returned linkedPerson in JSON response  
- ❌ UI showed linkedPerson after update but lost it after page refresh
- ❌ Sammanställning "Utbetalning" section showed no linked data

**Critical Fix Points**:
1. **budgetOrchestrator.ts:1949** - Add `linkedPerson: tx.linkedPerson || (tx as any).linked_person` to `forceReloadTransactions()`
2. **budgetOrchestrator.ts:3227** - Add `linkedPerson: tx.linkedPerson` to `setTransactionsForMonth()`  
3. **TransactionExpandableCard.tsx:69,112** - Add snake_case conversion in useState and useEffect
4. **types/transaction.ts:35** - Add `linkedPerson?: string` to ImportedTransaction interface
5. **types/budget.ts:27** - Add `linkedPerson?: string` to Transaction interface
6. **types/*.ts** - Add `Payment` to transaction type unions

**Additional Issues Found**:
- Missing `Users` icon import in `Sammanstallning.tsx:31` (runtime error)
- Multiple server instances causing request routing issues

**Prevention Strategy**: 
Always search for `.map(` functions when adding new fields to ANY entity. Use: `grep -rn "\.map.*=> ({" client/src/`

## Development Best Practices

- **Always use UUIDs**: Never string-based entity lookups
- **Orchestrator Pattern**: All mutations through `budgetOrchestrator.ts`
- **Real Data Testing**: Test imports with actual Swedish bank CSV/XLSX files
- **Payday Validation**: Verify 25th-of-month balance calculations
- **TanStack Query**: Use for all server state management
- **Mock Auth**: `dev-user-123` automatically injected in development

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Thedanielssson88) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
