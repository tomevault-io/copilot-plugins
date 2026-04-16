## vsol-admin

> VSol Admin is a local-first, SQLite-backed web application that replaces the Excel-based "golden sheet" for managing monthly consultant payroll cycles. The system automates calculations, provides audit logging, and maintains data integrity while preserving the familiar workflow.

# VSol Admin - Cursor Rules

## Project Context

VSol Admin is a local-first, SQLite-backed web application that replaces the Excel-based "golden sheet" for managing monthly consultant payroll cycles. The system automates calculations, provides audit logging, and maintains data integrity while preserving the familiar workflow.

Key features:
- Monthly payroll cycle management with automated line item creation
- Real-time calculation of subtotals and totals using Excel formula logic
- Snapshot rates at cycle creation to preserve historical data
- Audit logging for all state changes with user attribution
- Consultant CRUD with termination date support
- Invoice and payment tracking integrated with cycles
- Anomaly detection for missing data, bonuses without dates, etc.

Tech stack:
- Monorepo: Turborepo + pnpm workspaces
- Backend: Node.js + Express, Drizzle ORM (SQLite), Zod validation, bcrypt for auth
- Frontend: React + Vite, React Router v6, TanStack Query, TanStack Table, Shadcn/ui, date-fns
- Database: SQLite (`file:./dev.db`)
- Auth: JWT with username/password (users: rommel, isabel, celiane)

Architecture:
- Turborepo monorepo structure: `apps/api`, `apps/web`, `packages/shared`
- Backend in Express with Drizzle ORM for type-safe queries
- Frontend as React SPA with TanStack Query for server state
- Shared Zod schemas and TypeScript types in packages/shared
- SQLite file-based database (local-first)

## Code Style & Patterns

TypeScript:
- Use strict mode for type safety
- Prefer type inference when obvious
- Define explicit interfaces for API request/response
- Use Zod schemas for runtime validation (create schemas in packages/shared)
- Avoid `any` - use `unknown` and narrow with type guards
- Use discriminated unions for polymorphic data (e.g., Payment kinds)
- Export types from .d.ts files when used across packages

JavaScript patterns:
- Always use async/await over Promises chains
- Use pnpm workspaces for package management
- Create services for business logic, keep routes thin
- Use middleware for cross-cutting concerns (auth, logging, validation)
- Prefer functional composition over class inheritance
- Use Drizzle for all database operations (never raw SQL)

Naming conventions:
- Files: kebab-case (e.g., `cycle-service.ts`, `line-item-table.tsx`)
- Components: PascalCase (e.g., `GoldenSheetPage.tsx`)
- Functions: camelCase (e.g., `createPayrollCycle()`)
- Constants: UPPER_SNAKE_CASE (e.g., `JWT_SECRET`)
- Database tables: camelCase (e.g., `payrollCycles`, `cycleLineItems`)
- React hooks: prefix with `use` (e.g., `useCycleData()`)

Import/export:
- Use named exports for services and utilities
- Use default exports for React components
- Group imports: external → workspace packages → relative
- Absolute imports for shared package: `@vsol-admin/shared`
- Absolute imports for web: `@/components`, `@/lib`, `@/hooks`

What to avoid:
- Do not mutate database records directly - use service layer
- Do not store computed values in database - calculate on the fly
- Do not use index.ts exports with re-exports except for package boundaries
- Do not create cycles without auto-creating line items for active consultants
- Do not allow editing snapshotted ratePerHour in line items

## Technology-Specific Guidelines

Express/Node.js:
- Use Express Router for route organization by entity
- Validate request body with Zod schemas before service calls
- Use try-catch blocks in async route handlers
- Return 400 for validation errors, 401 for auth failures, 500 for server errors
- Use .env for secrets, never hardcode credentials
- Enable CORS only for localhost:5173 (web dev server)

Drizzle ORM:
- Define schema in `apps/api/src/db/schema.ts`
- Use relations() for joins (e.g., cycle.lines)
- Use eq() for WHERE clauses (eq(t.column, value))
- Always use transactions for multi-step operations (create cycle + lines)
- Use Drizzle's migration system for schema changes
- Return plain objects from DB, not Drizzle model instances

React + Vite:
- Use functional components only, no class components
- Use TanStack Query for all server state (useQuery, useMutation)
- Use React Router v6 for routing with data loaders
- Store form state in controlled components (controlled inputs)
- Use Shadcn/ui components from shadcn/ui library
- Use date-fns for date formatting and manipulation

TanStack Query:
- Create custom hooks in `apps/web/src/hooks/`
- Use optimistic updates for inline editing on Golden Sheet
- Cache keys: `['cycles', cycleId]`, `['lines', cycleId]`
- Use invalidateQueries after mutations to refresh UI
- Add retry logic only for createCycle mutations
- Use staleTime: 0 for real-time data (cycles, lines)

TanStack Table:
- Define column types explicitly for type safety
- Use meta objects to pass cycleId, consultantId to inline editors
- Implement cell renderers for each editable field type
- Use row grouping for readability on large tables
- Enable column sorting only for dashboard lists (not Golden Sheet)

Shadcn/ui:
- Use Button, Input, Card, Calendar for consistent UI
- Use Popover for date pickers in table cells
- Use Toast for success/error notifications
- Use Dialog for confirmation of destructive actions
- Style with Tailwind CSS classes
- Create custom components in `apps/web/src/components/ui/`

## Error Handling

Backend (Express):
- Create custom error classes in `apps/api/src/middleware/errors.ts`:
  - ValidationError (400) - Zod schema failures
  - NotFoundError (404) - entity not found
  - UnauthorizedError (401) - auth failures
- Wrap all route handlers in error handling middleware
- Log errors with context: userId, cycleId, request body
- Never expose database errors directly - return generic message
- Use Zod's .parse() for request validation

Frontend (React):
- Use React Error Boundary for page-level errors
- Use Toast notifications for mutation errors
- Use loading states for pending queries
- Use suspense boundaries for route data loaders
- Never silently catch errors - always show user feedback
- Log errors to console in development only

Database:
- Always use transactions for multi-step writes (create cycle + 10 line items)
- Catch SQLite constraint violations (foreign key, unique)
- Never delete consultants used in cycles - use soft delete
- Use ON DELETE CASCADE for relations only when appropriate (AuditLog → Cycle)
- Validate foreign keys before insert (e.g., consultantId exists)

Never silently catch:
- JWT signature verification failures
- Database transaction rollbacks
- Invalid date inputs (should be validated with Zod)
- Division by zero in calculations (workHours could be 0)

## Data & State Management

Database operations:
- Use transactions for all mutations that touch multiple tables
- Snapshot consultant rates in line items (ratePerHour) - do not reference Consultant.hourlyRate dynamically
- Use nullable fields for optional data (terminationDate, adjustmentValue)
- Use computed columns sparingly - prefer application-level calculations
- Index cycleId, consultantId columns for query performance

Query safety:
- Always use parameterized queries via Drizzle (no string concatenation)
- Validate foreign keys exist before creating relations
- Check for duplicates with UNIQUE constraints (consultant names)
- Use SELECT with explicit columns (not SELECT *)

Caching:
- Cache consultant list in memory (rarely changes)
- Cache cycle summary in memory for active cycle
- Do not cache computed totals - recalculate on every request
- Clear cache on cycle updates via TanStack Query invalidation

State management:
- Server state: TanStack Query (cycles, consultants, invoices)
- Form state: React useState for controlled inputs
- Auth state: React Context or Zustand for JWT token
- UI state: useState for dialogs, modals, dropdowns
- Do not use Redux - TanStack Query + Context is sufficient

Data validation:
- Validate all inputs with Zod schemas in packages/shared
- Sanitize user inputs before storing (no XSS in comments)
- Validate dates are within reasonable ranges (not year 1900)
- Validate currency amounts are positive numbers
- Validate consultant names are not empty strings

## Testing Requirements

What requires tests:
- All service methods (CycleService, ConsultantService)
- All API endpoints (happy path and error cases)
- Calculation functions (subtotal, totalHourlyValue, usdTotal)
- Audit logging (verify entries created on mutations)
- Authentication middleware (JWT verification, password hashing)

How to mock:
- Mock Drizzle client in service tests
- Mock TanStack Query in component tests
- Mock fetch API in API client tests
- Use MSW (Mock Service Worker) for integration tests
- Mock JWT signing/verification for auth tests

Test organization:
- Unit tests: `__tests__` folders co-located with files
- Integration tests: `tests/integration/` folder
- E2E tests: `tests/e2e/` folder (if using Playwright)
- Naming: `file-name.test.ts` or `file-name.spec.ts`
- Use Vitest for API tests, React Testing Library for components

Coverage expectations:
- 80% coverage for services and API routes
- 60% coverage for React components (focus on golden sheet interactions)
- 100% coverage for calculation functions (critical business logic)

Edge cases to always consider:
- Creating cycle with no active consultants (should error)
- Updating line item when cycle is locked (if adding locking feature)
- Editing consultant rate after cycle created (should not affect existing cycles)
- Negative bonusAdvance or adjustmentValue (allowed by business rules)
- Date inputs outside valid ranges (should be caught by Zod)
- Concurrent updates to same cycle (handle with optimistic updates)

## Documentation Standards

When to update docs:
- Adding new API endpoint → update API docs comment
- Creating new service method → add JSDoc comment
- Changing business logic → update plan document
- Adding new route → add route description
- Changing environment variables → update .env.example

Where docs go:
- Code comments: JSDoc for public functions, inline comments for complex logic
- API documentation: OpenAPI spec in `apps/api/src/routes/`
- README: Setup instructions, environment variables, dev workflow
- Plan document: Business logic, formulas, workflows
- ADR (Architecture Decision Records): For major technical decisions

Code comment philosophy:
- Comment the "why" not the "what"
- Document business rules (e.g., "only active consultants are included in new cycles")
- Comment calculations that match Excel formulas (with formula reference)
- Explain anomaly detection logic
- Document API endpoints with request/response examples
- Do not comment obvious code (e.g., `const x = 1; // set x to 1`)

README maintenance:
- Keep "Getting Started" instructions current
- Update environment variables section as needed
- Document new scripts added to package.json
- Include troubleshooting for common setup issues
- Link to detailed plan document

## File Organization

Directory structure:
```
vsol-admin/
├── apps/
│   ├── api/                    # Express API server
│   │   ├── src/
│   │   │   ├── server.ts       # Entry point, Express app setup
│   │   │   ├── routes/         # REST endpoints (by entity)
│   │   │   │   ├── auth.ts
│   │   │   │   ├── consultants.ts
│   │   │   │   ├── cycles.ts
│   │   │   │   ├── invoices.ts
│   │   │   │   └── payments.ts
│   │   │   ├── services/       # Business logic
│   │   │   │   ├── cycle-service.ts
│   │   │   │   ├── consultant-service.ts
│   │   │   │   ├── invoice-service.ts
│   │   │   │   ├── payment-service.ts
│   │   │   │   └── audit-service.ts
│   │   │   ├── middleware/     # Auth, validation, error handling
│   │   │   │   ├── auth.ts
│   │   │   │   ├── validate.ts
│   │   │   │   └── errors.ts
│   │   │   ├── db/             # Drizzle schema and client
│   │   │   │   ├── schema.ts
│   │   │   │   ├── index.ts    # DB client export
│   │   │   │   └── seed.ts
│   │   │   └── lib/            # Utilities
│   │   │       ├── jwt.ts
│   │   │       └── bcrypt.ts
│   │   ├── drizzle/            # Drizzle migrations
│   │   ├── scripts/
│   │   │   └── backup-db.js
│   │   └── package.json
│   └── web/                   # React SPA
│       ├── src/
│       │   ├── main.tsx        # Entry point
│       │   ├── App.tsx         # Router setup
│       │   ├── routes/         # Page components
│       │   │   ├── login.tsx
│       │   │   ├── dashboard.tsx
│       │   │   ├── golden-sheet.tsx
│       │   │   ├── consultants.tsx
│       │   │   ├── invoices.tsx
│       │   │   ├── payments.tsx
│       │   │   └── audit.tsx
│       │   ├── components/     # Reusable UI
│       │   │   ├── ui/         # Shadcn components
│       │   │   └── cycle/
│       │   ├── lib/            # API client, utils
│       │   │   ├── api-client.ts
│       │   │   └── auth.ts
│       │   ├── hooks/          # React Query hooks
│       │   │   ├── use-cycles.ts
│       │   │   ├── use-consultants.ts
│       │   │   └── use-cycle-data.ts
│       │   └── contexts/
│       │       └── auth-context.tsx
│       └── package.json
├── packages/
│   └── shared/                 # Shared types and schemas
│       └── src/
│           ├── types.ts
│           ├── schemas.ts
│           └── index.ts
├── turbo.json
├── package.json                # Root workspace
└── pnpm-workspace.yaml
```

Where new code goes:
- New API route → `apps/api/src/routes/`
- New service logic → `apps/api/src/services/`
- New page → `apps/web/src/routes/`
- New component → `apps/web/src/components/`
- New shared type → `packages/shared/src/types.ts`
- New shared schema → `packages/shared/src/schemas.ts`
- New React hook → `apps/web/src/hooks/`
- New utility function → `apps/api/src/lib/` or `apps/web/src/lib/`

Co-location rules:
- Component tests next to component file (same folder)
- Service tests co-located with service files
- Shadcn components in `apps/web/src/components/ui/`
- Page-specific components in route folder (e.g., `routes/golden-sheet/`)

## Security & Privacy

Credential management:
- Store passwords as bcrypt hashes (never plaintext)
- Use JWT tokens with 24-hour expiration
- Store JWT_SECRET in .env (never commit)
- Use HTTP-only cookies for tokens (optional security enhancement)
- Do not log request bodies containing passwords

Input sanitization:
- Sanitize all text inputs (comments, consultant names) to prevent XSS
- Validate email formats if adding email fields
- Use Zod schemas for all input validation
- Reject input with null bytes or special SQL characters
- Trim whitespace from string inputs

Sensitive data handling:
- Never log audit log contents in production (contains potentially sensitive data)
- Hash user passwords before storage (bcrypt with 10 rounds)
- Do not store credit card information (out of scope)
- Encrypt backups if storing on shared drives
- Use parameterized queries (Drizzle handles this automatically)

What never to commit:
- `.env` files (add to .gitignore)
- JWT_SECRET values
- Database files (dev.db should be in .gitignore)
- Real passwords or credentials
- Personal or sensitive business data

Privacy considerations:
- Audit logs capture user IDs - ensure JWT tokens are not exposed in logs
- User data is minimal (username, role only)
- Consultant data is business information (names, rates)
- Cycle data contains payment information - protect database file
- No PII beyond consultant names (no addresses, SSN, etc.)

## Performance Guidelines

Batching strategies:
- Batch line item creation when creating cycle (all 10 in single transaction)
- Use Drizzle's batch insert for seeding data
- Batch TanStack Query invalidations (invalidate multiple queries at once)
- Avoid N+1 queries (use eager loading with relations)

Rate limiting:
- Not needed for MVP (local-first app)
- Future: Add rate limiting if opening to internet
- Track API usage via audit logs for monitoring

Caching policies:
- Cache consultant list in React Query (staleTime: 5 minutes)
- Cache cycle data with staleTime: 1 minute (for active editing)
- Do not cache computed totals - always recalculate
- Clear cache on mutations via invalidation
- Use Drizzle's query cache for repeated queries (automatic)

Resource cleanup:
- Close database connections on server shutdown
- Unmount React Query subscriptions on component unmount
- Clear intervals/timeouts in useEffect cleanup
- Close file streams after backups

When to paginate:
- Consultant list (future: >100 consultants)
- Audit log viewer (future: >1000 entries)
- Cycle list: load all (typically <50 cycles)
- Do not paginate Golden Sheet page (always shows all consultants)

## Common Pitfalls

1. **Mutating snapshotted data**: Do not allow editing ratePerHour in line items after cycle creation - it's a historical snapshot

2. **Including terminated consultants**: When creating new cycle, check Consultant.terminationDate is null (only active consultants)

3. **Forgetting to recompute totals**: When updating line item, trigger cycle recomputation (totalHourlyValue changes)

4. **Not handling null dates**: UI must handle nullable date fields (bonusDate, informedDate) - do not assume dates always exist

5. **Race conditions in optimistic updates**: TanStack Query handles this, but be careful when editing same field twice rapidly

6. **SQLite constraints**: If consultant is deleted while used in cycle line items, you'll get foreign key violation - implement soft delete

7. **Date format mismatches**: Use ISO 8601 format everywhere (YYYY-MM-DD) - date-fns handles conversion

8. **Calculation discrepancies**: Ensure usdTotal matches Excel formula exactly: `=B22*B26-(B23+B24)+B25+B27`

9. **Audit log size**: After 6 months of cycles, audit log table could be large - implement cleanup or archiving

10. **Missing validation**: Always validate foreign key references (consultantId, cycleId exist) before inserts

## Change Checklist

When making changes, consider:
- [ ] Does similar functionality exist? (search codebase first)
- [ ] Will this break existing cycles? (backward compatibility)
- [ ] Are tests updated? (add tests for new logic)
- [ ] Is documentation updated? (update plan or README)
- [ ] Does this need manual testing? (test the affected workflow)
- [ ] Are audit logs created? (for all mutations)
- [ ] Do I need to update Zod schemas? (for new fields)
- [ ] Will calculation logic change? (verify totals still match Excel)
- [ ] Are foreign key constraints satisfied? (validate references exist)
- [ ] Does this affect seed data? (update seed.ts if needed)

## System-Specific Features

### Creating a Payroll Cycle

Where: `apps/api/src/services/cycle-service.ts` - `createCycle(monthLabel, globalWorkHours, omnigoBonus)`

How it works:
- Creates PayrollCycle record with header dates and footer values
- Queries all consultants WHERE terminationDate IS NULL
- For each active consultant, creates CycleLineItem with:
  - consultantId (reference to consultant)
  - ratePerHour (snapshot of Consultant.hourlyRate at this moment)
  - workHours (null, uses cycle.globalWorkHours as default)
  - All other fields default to null or false
- Returns cycle with populated lines[]

Edge cases:
- No active consultants → return error (cannot create empty cycle)
- Consultant has null hourlyRate → use 0.00 (should not happen, but safe default)
- Duplicate monthLabel → enforce uniqueness in database schema

Configuration: globalWorkHours, omnigoBonus optional on create

When to modify: If adding new line item fields or changing consultant selection logic

Integration: Triggers audit log entry with action="CREATE_CYCLE"

### Calculating Subtotal for Line Item

Where: `apps/api/src/services/cycle-service.ts` - computed in `getCycleSummary(cycleId)`

How it works:
```typescript
const workHours = lineItem.workHours || cycle.globalWorkHours;
const rateAmount = workHours * lineItem.ratePerHour;
const adjustment = lineItem.adjustmentValue || 0;
const advance = lineItem.bonusAdvance || 0;
const subtotal = rateAmount + adjustment - advance;
```

Formula: `subtotal = (workHours * ratePerHour) + adjustmentValue - bonusAdvance`

Edge cases:
- workHours is null → use cycle.globalWorkHours
- ratePerHour is 0 → subtotal is negative if adjustments exist (allowed by business)
- bonusAdvance > base amount → subtotal becomes negative (should flag as anomaly)

Configuration: lineItem.workHours overrides cycle.globalWorkHours if not null

When to modify: When business rules change for calculating consultant payment

Integration: Used in Golden Sheet table display, included in cycle summary

### Calculating USD Total for Cycle

Where: `apps/api/src/services/cycle-service.ts` - computed in `getCycleSummary(cycleId)`

How it works:
```typescript
// Base amount = sum of each consultant's individual work hours * their rate
const baseAmount = SUM(for each line item: (workHours || globalWorkHours) * ratePerHour);
const paymentSubtractions = cycle.pagamentoPIX + cycle.pagamentoInter;
const bonuses = cycle.omnigoBonus + cycle.equipmentsUSD;
const usdTotal = baseAmount - paymentSubtractions + bonuses;
```

Formula uses individual consultant work hours (not a single global value):
- For each consultant: uses their specific `workHours` if set, otherwise falls back to `globalWorkHours`
- Base amount = sum of all (consultant work hours × consultant rate)
- Then subtract payments (PIX + Inter) and add bonuses (Omnigo + Equipment)

Edge cases:
- If a consultant has individual workHours set, it overrides globalWorkHours for that consultant
- If globalWorkHours is 0 and no individual workHours set → that consultant contributes 0 to base amount
- Negative payments → results in positive addition (should not happen but safe)

Configuration: All input fields editable on Golden Sheet page

When to modify: If Excel formula changes or new adjustment fields added

Integration: Displayed on Dashboard KPI card and Golden Sheet footer

### Auditor Logging System

Where: `apps/api/src/services/audit-service.ts` + middleware

How it works:
- Every mutation (POST, PUT, DELETE) triggers audit log entry
- Captures: userId (from JWT), action (e.g., "UPDATE_LINE_ITEM"), entityType, entityId, changes (JSON diff of old vs new values), timestamp
- Stored in audit_log table
- Accessible via `GET /audit?cycleId=X&userId=Y`

Edge cases:
- Bulk updates (create cycle with 10 lines) → one log entry for cycle, not 10 for lines
- Deleted entity → audit log entry still exists with entityId
- Failed mutations → do not create audit log entry

Configuration: Enabled for all routes except GET requests

When to modify: When adding new entity types or changing change tracking logic

Integration: Used for traceability, troubleshooting, and compliance

### Golden Sheet Inline Editing

Where: `apps/web/src/routes/golden-sheet.tsx` + TanStack Table

How it works:
- TanStack Table with column definitions for each field
- Editable cells rendered with input components (text, number, date picker)
- User clicks cell → opens inline editor (replaces cell content)
- User edits value → onBlur or Enter key triggers save
- Optimistic update via React Query mutation
- On success: UI reflects new value
- On error: revert optimistic update + show Toast error

Edge cases:
- Two users editing same field simultaneously → last-write-wins (optimistic update)
- Date picker in small table cell → use Popover with Calendar component
- Number inputs for currency → format as "$0.00" but store as number
- Long comments → use textarea cell renderer

Configuration: All fields editable except Consultant Name and Rate per Hour

When to modify: If adding new editable columns or changing validation

Integration: Mutations trigger cycle recomputation, updates refresh table

## Configuration & Infrastructure

Key configuration files:
- `turbo.json` - Build pipeline configuration, task dependencies
- `apps/api/drizzle.config.ts` - Drizzle ORM configuration (SQLite connection string)
- `apps/web/vite.config.ts` - Vite bundler configuration, path aliases
- `package.json` (root) - Workspace setup, dev scripts
- `pnpm-workspace.yaml` - pnpm workspace definitions

Environment variables (`.env` files):
- `apps/api/.env`:
  - `PORT` (default: 3000) - API server port
  - `JWT_SECRET` - Secret for signing JWT tokens (required)
  - `DATABASE_URL` (default: file:./dev.db) - SQLite connection string
  - `NODE_ENV` (development/production) - Controls logging level
- `apps/web/.env`:
  - `VITE_API_URL` (default: http://localhost:3000/api) - Backend API base URL
  - (Optional) `VITE_APP_NAME` - Application display name

Build/deployment:
- Development: `pnpm dev` runs both API (port 3000) and web (port 5173) concurrently
- Production build: `pnpm build` creates optimized bundles
- API builds to `apps/api/dist/` (Drizzle migrations included)
- Web builds to `apps/web/dist/` (static files for hosting)
- Database migrations: run with `pnpm db:migrate` before starting API

Development vs production:
- Development: Hot module reloading, verbose error messages, unminified code
- Production: Minified bundles, error boundaries, disabled dev tools
- Database: dev.db for development, production.db for production (different files)
- Logging: Console logs in dev, structured logs in production (future)
- CORS: Open in dev (localhost:*), restricted in production

External service configurations:
- Wave Apps API (future) - Wave accounting integration credentials
- Time Doctor API (future) - TD integration for hours tracking
- Payoneer API (future) - Bulk payment export credentials
- Email service (future) - SMTP configuration for notifications

Database backup:
- Script: `apps/api/scripts/backup-db.js`
- Run with: `node scripts/backup-db.js`
- Creates timestamped backup in `backups/` directory
- Should run before schema migrations or data migrations
- Manual process for MVP (no auto-scheduling)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DebonairSM) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
