## couple-time

> **ALWAYS** read `docs/HOW-TO-USE.md` first to understand the project documentation system and current status.

## 📖 Getting Started
**ALWAYS** read `docs/HOW-TO-USE.md` first to understand the project documentation system and current status.

---

## 1. Authentication & Authorization (App Router)

- Use centralized `middleware.ts` for all protected routes
- **Do not** use `withAuth` wrappers
- Use `matcher` in `middleware.ts` to define protected routes
- Avoid permission logic inside individual route handlers


## 2. API Route Standards
- Use consistent URL patterns for all API routes
- All protected routes should use centralized permission middleware
- Combine related operations into single files when possible
- Use standardized response format: { success: boolean, data?: any, error?: { message: string } }
- Group routes by permissions (auth, admin, coordinator, super, public etc.)
- Avoid checking permissions inside each function separately - use middleware
- Use pagination only after approval. This will probably only be in the admin API, in the other requests a relatively small amount of data is expected, so all the logic will be done in the client.

For each API endpoint:
1. ✅ Unit tests:
   - Does the handler behave correctly with valid input?
   - Are all edge cases covered?
   - Are expected success and failure cases tested?

2. ✅ Integration tests (if applicable):
   - Does the handler integrate properly with authentication middleware?
   - Are permissions and role-based access enforced correctly?
   - Are side effects (e.g. DB updates) verified?
   - Are invalid tokens / no session handled with proper status codes?

3. ✅ Error handling:
   - Is a test written for every thrown error or invalid input?
   - Is the error format `{ success: false, error: { message } }` verified?

4. ✅ Response format:
   - Are success responses tested to follow `{ success: true, data }` format?
   - Are responses validated with `expect.objectContaining(...)` to avoid brittle tests?

5. ✅ Coverage:
   - Are all branches, conditionals, and early returns tested?

6. ✅ Mocking:
   - Are external dependencies (e.g., Prisma, getToken) properly mocked to isolate logic?


---

## 3. Testing Standards

- Use **`vitest` only** for all unit/integration tests
- Shared utilities → `src/test/utils.ts`
- Test factories → `src/test/factories.ts`

### 📁 Test File Naming
- Use `*.test.ts` suffix
- Place close to the tested logic when practical

## 4. UI & Data Handling

---

- No data fetching inside UI components unless absolutely necessary
- Each feature/screen must be designed before coding
- Avoid generic or over-abstracted components unless reused
- Use elements from /components, ןncluding but not limited to icons, buttons, fields, etc.
- If a component can be used more than once, save it in a separate file in the appropriate context.
- Track the required information, and if necessary, request changes to stores and APIs to streamline processes. Don't make big changes all at once.
- Place a high emphasis on speed, load the information in the background, and show the user as many things as possible that don't require the data being loaded. Show loading signs where necessary.

---

## 5. Code Quality & Maintainability

- Keep code readable without extra context
- Don’t abstract core logic away into hidden files
- Log clearly – especially around auth or data failures
- Only optimize when needed

---

## 6. Cleanup Policy

- Regularly remove unused code, mocks, or placeholder files
- Don’t leave commented-out code in the repo
- All files must have a clear reason to exist, or be deleted

## 7. Types Architecture

### 📁 Type File Structure
Our types follow a practical, business-focused approach organized in layers:

```
/types/
├── schema.ts       # Base Prisma types (User, Center, Game, etc.)
├── models.ts       # Query objects + generated types for stores/UI
├── computed.ts     # Business logic enhancements
├── api.ts          # Request contracts only
└── index.ts        # Central exports
```

### ✅ Type File Purposes

**`schema.ts`** - Foundation Layer
- Re-exports base Prisma model types (`User`, `Center`, `Game`, etc.)
- Re-exports enums (`Role`, `Area`, `GameCategories`, etc.)
- Source of truth for database entities

**`models.ts`** - Data Models Layer
- Query objects for database operations (`USER_CONTACT_FIELDS`, `RENTAL_FOR_USER`)
- Generated TypeScript types for stores/UI (`UserContact`, `RentalForUser`)
- Clean composition using select/include patterns

**`computed.ts`** - Business Logic Layer
- Enhanced types with computed fields (`RentalWithDetails`, `CenterWithStats`)
- Context-specific business logic (users vs coordinators see different data)
- Only reusable business enhancements, not single-use API responses

**`api.ts`** - Request Contracts Layer
- Request payload types only (`CreateGameRequest`, `UpdateUserRequest`)
- Filter types (`GameFilters`, `CenterFilters`)
- Responses use `ApiResponse<T>` from `/lib/api-response.ts`

### ✅ Type Usage Patterns

**In API Routes:**
```typescript
// Use query objects for database operations
const rentals = await prisma.rental.findMany(RENTAL_FOR_USER);
const centers = await prisma.center.findMany({ select: CENTER_BASIC_FIELDS });

// Use request types for validation
const body = CreateGameRequest.parse(await request.json());

// Return with ApiResponse wrapper
return apiResponse(true, computedData);
```

**In Stores/UI:**
```typescript
// Use generated types for state management
const [rental, setRental] = useState<RentalForUser | null>(null);
const [centers, setCenters] = useState<CenterWithCoordinator[]>([]);

// Use computed types for enhanced business logic
const enhancedRental: RentalWithDetails = {
  ...rental,
  isOverdue: computeOverdue(rental),
  canCancel: computeCanCancel(rental)
};
```

### ❌ Anti-Patterns to Avoid
- Don't create interfaces for single-use API responses
- Don't duplicate types that can be generated from query objects
- Don't put business logic in the basic model types
- Don't create over-engineered select/include patterns unless actually used

### ✅ Type Naming Conventions
- **Base models**: `User`, `Center`, `Game` (from Prisma)
- **Query objects**: `USER_CONTACT_FIELDS`, `RENTAL_FOR_USER` (SCREAMING_SNAKE_CASE)
- **Generated types**: `UserContact`, `RentalForUser` (PascalCase)
- **Enhanced types**: `RentalWithDetails`, `CenterWithStats` (descriptive suffixes)
- **Context-specific**: `ForUser`, `ForCoordinator` (shows perspective)

---

## Development Standards Summary

- ✅ Follow documentation workflow in `docs/HOW-TO-USE.md`
- ✅ Keep code minimal and focused
- ✅ Import types from `/types` architecture
- ✅ Write comprehensive tests for all endpoints
- ✅ Use standardized response format
- ✅ Clean up unused code regularly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tamarLevanoni) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
