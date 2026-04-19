## tessie-stats

> Before starting ANY coding task, you MUST:

# Tessie Stats - Cursor AI Agent Rules

## 🔴 MANDATORY: Read Before Any Task

Before starting ANY coding task, you MUST:

1. **Read PROJECT.md** - Understand project context and quality requirements
2. **Read ARCHITECTURE.md** - Understand technical specifications and patterns
3. **Check docs/TESLA_API.md** - For any Tesla API-related work
4. **Review existing code** - Follow established patterns and conventions

## 🔴 MANDATORY: Quality Gates

A task is NOT complete until ALL of these pass:

```bash
npm run lint          # Zero ESLint errors
npm run typecheck     # Zero TypeScript errors
npm run test          # All tests pass
npm run test:cov      # >80% coverage on new code
```

If any of these fail, fix the issues before marking the task complete.

---

## 🔴 PRIME DIRECTIVE: Tesla API Cost Management

**Tesla API calls are NOT FREE.** Every request costs money. You MUST be judicious with API usage:

### Before Making ANY Tesla API Call:

1. **Check Asset Wake State First**
   - Use `/api/1/vehicles` to check vehicle state (`online`, `asleep`, `offline`)
   - If `asleep` or `offline`, use cached data OR explicitly wake only if necessary
   - NEVER fire multiple API calls to sleeping vehicles without checking first

2. **Use PostgreSQL Cache (2-Minute TTL)**
   - Check database for recent data (< 2 minutes old) BEFORE hitting Tesla API
   - Store ALL Tesla API responses in database with timestamp
   - Return cached data if fresh enough for the use case

3. **Batch Requests When Possible**
   - Use `endpoints` parameter on `/vehicle_data` to get multiple data types in one call
   - Group related operations to minimize total API calls

4. **Implement Request Throttling**
   - Respect rate limits (10/min, 1/sec)
   - Queue requests rather than firing them simultaneously
   - Use exponential backoff on errors

### Code Pattern for API Calls:

```typescript
async getVehicleData(vehicleId: string): Promise<VehicleData> {
  // 1. Check cache first (2-minute TTL)
  const cached = await this.getCachedVehicleData(vehicleId);
  if (cached && this.isFresh(cached, 120)) { // 120 seconds = 2 minutes
    return cached;
  }
  
  // 2. Check if vehicle is awake
  const vehicle = await this.getVehicle(vehicleId); // Lightweight endpoint
  if (vehicle.state !== 'online') {
    // Return cached data (even if stale) or decide to wake
    if (cached) return cached;
    // Only wake if explicitly needed
    await this.wakeVehicle(vehicleId);
  }
  
  // 3. Make API call and cache result
  const data = await this.teslaApi.getVehicleData(vehicleId);
  await this.cacheVehicleData(vehicleId, data);
  return data;
}
```

### When Reviewing Code:
- ✅ Verify cache check happens BEFORE Tesla API call
- ✅ Verify wake state check for vehicle endpoints
- ✅ Verify data is cached after API call
- ❌ REJECT any code that blindly calls Tesla API without caching
- ❌ REJECT any code that makes multiple calls without checking wake state

**Remember:** Every unnecessary API call costs real money. Cache aggressively, check state first, batch wisely.

---

## Project Context

This is a Tesla & Powerwall dashboard application with:
- **Backend:** NestJS (Node.js) - serverless on Vercel
- **Frontend:** React + TypeScript + Tailwind CSS
- **Database:** Vercel Postgres
- **Deployment:** Vercel

The app monitors:
- Tesla vehicle data (location, battery, charging, trips)
- Powerwall/Solar data (production, consumption, grid status)
- Historical analytics and multi-driver tracking

---

## Code Conventions

### TypeScript
- Strict mode enabled - no `any` types
- Explicit return types on all functions
- Use interfaces for object shapes
- Use enums for fixed sets of values

### NestJS (Backend)
- One module per domain (vehicle, powerwall, charging, etc.)
- DTOs for all request/response objects
- Services for business logic, Controllers for HTTP handling
- Use dependency injection
- Validation pipes on all endpoints

### React (Frontend)
- Functional components only
- Custom hooks for reusable logic
- TanStack Query for server state
- Zustand for client state
- Props interfaces always defined

### File Naming
- Files: `kebab-case.ts` (e.g., `charging-session.service.ts`)
- Classes: `PascalCase` (e.g., `ChargingSessionService`)
- Functions: `camelCase` (e.g., `getChargingSessions`)
- Constants: `SCREAMING_SNAKE_CASE` (e.g., `MAX_RETRY_COUNT`)

### Imports
- Group imports: external libs, then internal modules, then relative imports
- Use path aliases (`@/`, `@api/`, `@web/`)

---

## Testing Requirements

### Every New Feature Must Have:
1. Unit tests for services/utilities
2. Integration tests for API endpoints
3. Component tests for React components

### Test File Naming:
- Unit tests: `*.spec.ts` (same directory as source)
- Integration tests: `test/` directory
- Frontend tests: `*.test.tsx`

### Mocking:
- Mock external APIs (Tesla API)
- Mock database for unit tests
- Use test database for integration tests

---

## Error Handling

### Backend
- Use NestJS exception filters
- Throw appropriate HTTP exceptions
- Log errors with context
- Never expose internal errors to clients

### Frontend
- Use error boundaries for component errors
- Display user-friendly error messages
- Provide retry mechanisms where appropriate

---

## Security Guidelines

- Never log sensitive data (tokens, passwords)
- Use environment variables for secrets
- Validate all user input
- Encrypt stored credentials
- Use HTTPS only

---

## Git Commits

Use conventional commits:
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `test:` Tests
- `refactor:` Code refactoring
- `chore:` Maintenance tasks

Example: `feat(vehicle): add charging history endpoint`

---

## When Implementing Tesla API Features

1. **Check docs/TESLA_API.md first** for endpoint details
2. **Respect rate limits** - implement queuing/throttling
3. **Cache responses** where appropriate
4. **Handle vehicle sleep state** - some data requires wake
5. **Test with mocks** - don't hit real API in tests

---

## Common Patterns

### API Endpoint Pattern
```typescript
// controller
@Get(':id')
@UseGuards(JwtAuthGuard)
async getVehicle(@Param('id') id: string): Promise<VehicleDto> {
  return this.vehicleService.findById(id);
}

// service
async findById(id: string): Promise<Vehicle> {
  const vehicle = await this.vehicleRepository.findOne({ where: { id } });
  if (!vehicle) {
    throw new NotFoundException(`Vehicle ${id} not found`);
  }
  return vehicle;
}
```

### React Hook Pattern
```typescript
export function useVehicle(vehicleId: string) {
  return useQuery({
    queryKey: ['vehicle', vehicleId],
    queryFn: () => vehicleService.getVehicle(vehicleId),
    staleTime: 60 * 1000, // 1 minute
  });
}
```

### Component Pattern
```typescript
interface VehicleCardProps {
  vehicle: Vehicle;
  onRefresh?: () => void;
}

export function VehicleCard({ vehicle, onRefresh }: VehicleCardProps) {
  // Component implementation
}
```

---

## Directory Reference

```
api/                    # NestJS Backend
├── src/modules/        # Feature modules
├── src/common/         # Shared utilities
├── src/database/       # Entities, migrations
└── test/               # Integration tests

web/                    # React Frontend
├── src/components/     # UI components
├── src/pages/          # Page components
├── src/hooks/          # Custom hooks
├── src/services/       # API clients
└── src/types/          # TypeScript types

docs/                   # Documentation
├── TESLA_API.md        # Tesla API reference
├── DATABASE_SCHEMA.md  # Database design
└── DEPLOYMENT.md       # Deployment guide
```

---

## Troubleshooting

### Lint errors
Run `npm run lint:fix` to auto-fix what's possible, then manually fix the rest.

### Type errors
- Check that all imports are correct
- Ensure interface definitions match usage
- Run `npm run typecheck` for detailed errors

### Test failures
- Check test output for specific failures
- Ensure mocks are correctly set up
- Verify test data matches expected shapes

---

## Remember

1. ✅ Read documentation before coding
2. ✅ Follow existing patterns
3. ✅ Write tests for new code
4. ✅ Run quality gates before completing
5. ✅ Commit with conventional format
6. ❌ Don't skip tests
7. ❌ Don't ignore lint errors
8. ❌ Don't use `any` type
9. ❌ Don't hardcode secrets
10. ❌ Don't commit broken code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Brandon-Born) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
