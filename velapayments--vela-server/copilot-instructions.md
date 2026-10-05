## unit-tests

> Unit test patterns, conventions and commands for core-api


# Unit Test Conventions — Vela

Tests use **Jest 30** with **@nestjs/testing**. Co-located next to their source files.

## Useful Commands

Run all commands from the root directory:

```bash
# Run ALL unit tests
npx jest

# Run tests for a specific module
npx jest --testPathPatterns users
npx jest --testPathPatterns bounties
npx jest --testPathPatterns auth/guards

# Run a single test file
npx jest --testPathPatterns users.service.spec

# Run tests in watch mode (re-run on file changes)
npx jest --watch

# Run tests with coverage report
npx jest --coverage

# Run coverage for a single module
npx jest --coverage --testPathPatterns users

# Run tests matching a specific test name
npx jest -t "should throw NotFoundException"
```

Or use the standard npm scripts:

```bash
npm run test
npm run test:cov
```

## Coverage Configuration

Configured in `package.json` under `jest`:

- **Thresholds**: 75% statements, 75% lines, 75% functions, 70% branches
- **Collected from**: `*.service.ts`, `*.controller.ts`, `*.guard.ts`, `*.interceptor.ts`, `*.pipe.ts`, `*.filter.ts`
- **Excluded**: `*.spec.ts`, `*.e2e-spec.ts`, `auth/strategies/**` (Supabase strategy depends on external SDK)

When adding a new module, ensure tests meet the global thresholds before pushing.

## File Location & Naming

Tests live **next to** the file they test:

```
modules/users/
├── users.service.ts
├── users.service.spec.ts
├── users.controller.ts
├── users.controller.spec.ts
```

## Test Structure

```typescript
describe('FeatureService', () => {
  let service: FeatureService;

  const mockPrismaService = {
    model: { findUnique: jest.fn(), create: jest.fn(), update: jest.fn() },
  };
  const mockEventEmitter = { emit: jest.fn() };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        FeatureService,
        { provide: PrismaService, useValue: mockPrismaService },
        { provide: EventEmitter2, useValue: mockEventEmitter },
      ],
    }).compile();

    service = module.get<FeatureService>(FeatureService);
  });

  afterEach(() => jest.clearAllMocks());
});
```

## Controller Tests vs Service Tests

- **Controller tests**: verify delegation to service, correct param passing, no business logic duplication. Mock the entire service.
- **Service tests**: verify full business logic — happy path + every exception path. Mock Prisma and EventEmitter.

## Key Patterns

- Mock ALL dependencies — never use real DB or external services
- Test every exception: `NotFoundException`, `ForbiddenException`, `BadRequestException`, `ConflictException`
- Verify `eventEmitter.emit` calls with correct event name and payload
- Verify Prisma methods called with correct `where`/`data`/`include`
- Use `mockResolvedValue` / `mockRejectedValue` for async mocks
- Test idempotency where applicable (e.g., duplicate role additions)
- Use `expect.any(Array)` or `expect.objectContaining()` for flexible assertions

## Naming Convention

```typescript
it('should throw NotFoundException when user does not exist', ...)
it('should create application with AWAITING_ASSIGNMENT status', ...)
it('should emit user.created event after creating user', ...)
it('should return 403 when non-admin tries to add role', ...)
```

---
> Source: [VelaPayments/vela-server](https://github.com/VelaPayments/vela-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
