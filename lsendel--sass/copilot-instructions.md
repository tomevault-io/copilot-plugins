## sass

> providesTags: ["Payment"],

# AI Coding Agent Instructions

## Project Overview

Enterprise payment platform using Spring Boot Modulith (backend), React/TypeScript (frontend), and Python tools for development enforcement. Key components:

- `backend/`: Spring Boot modular monolith (Java 21, Spring Boot 3.5)
- `frontend/`: React + Vite + TypeScript web app
- `src/`: Python utilities and enforcement agents
- `tools/`: Constitutional development tools

## Critical Patterns

### Modular Architecture

#### Module Boundaries

- Each module is a self-contained unit in `backend/src/main/java/com/sass/`
- Modules only communicate through well-defined events and interfaces
- Example module structure:
  ```
  backend/payment/
  ├── api/                 # Public interfaces and DTOs
  │   ├── PaymentRequest.java
  │   └── PaymentService.java
  ├── internal/           # Implementation details (private)
  │   ├── StripeAdapter.java
  │   └── PaymentProcessor.java
  ├── events/            # Event definitions
  │   ├── PaymentCompletedEvent.java
  │   └── PaymentFailedEvent.java
  └── PaymentModuleConfiguration.java
  ```

#### Event Communication Examples

1. Payment Processing Flow:

   ```java
   // PaymentService emits event
   @Service
   class PaymentServiceImpl {
       @Autowired
       private ApplicationEventPublisher events;

       @Transactional
       public void processPayment(PaymentRequest request) {
           // Process payment...
           events.publishEvent(new PaymentCompletedEvent(paymentId));
       }
   }

   // Subscription module listens for payment events
   @Service
   class SubscriptionManager {
       @EventListener
       public void onPaymentCompleted(PaymentCompletedEvent event) {
           // Update subscription status...
       }
   }
   ```

2. User Activity Tracking:

   ```java
   // User module emits events
   @Service
   class UserServiceImpl {
       @Autowired
       private ApplicationEventPublisher events;

       public void updateProfile(ProfileUpdateRequest request) {
           // Update profile...
           events.publishEvent(new UserProfileUpdatedEvent(userId));
       }
   }

   // Audit module captures all user events
   @Service
   class AuditLogger {
       @EventListener
       public void logUserActivity(UserProfileUpdatedEvent event) {
           // Log user activity with correlation ID...
       }
   }
   ```

Key Rules:

- Modules never import from other modules' `internal` packages
- Events are immutable and contain only essential data
- Event handlers are idempotent and handle failures gracefully

### Development Workflow

1. Always start infrastructure first:

```bash
docker compose up -d  # Postgres, Redis, Mailhog, Adminer
```

2. Backend development:

```bash
cd backend && ./gradlew bootRun  # Start Spring Boot
./gradlew test jacocoTestReport  # Run tests with coverage
```

3. Frontend development:

```bash
cd frontend
npm install && npm run dev      # Development server
npm test                       # Unit tests
npm run test:e2e              # E2E tests (requires test:e2e:install first)
```

### Test-Driven Development Workflow

#### 1. Write the Test First

```java
// PaymentServiceTest.java
@Test
void shouldDeclinePaymentWhenInsufficientFunds() {
    // Given
    var payment = new PaymentRequest("user123", BigDecimal.valueOf(100));
    when(stripeClient.checkBalance("user123")).thenReturn(BigDecimal.valueOf(50));

    // When
    var result = paymentService.processPayment(payment);

    // Then
    assertThat(result.getStatus()).isEqualTo(PaymentStatus.DECLINED);
    assertThat(result.getReason()).isEqualTo("Insufficient funds");
}
```

#### 2. Run Tests (They Should Fail)

```bash
./gradlew test --tests PaymentServiceTest
```

#### 3. Implement the Feature

```java
@Service
class PaymentService {
    public PaymentResult processPayment(PaymentRequest payment) {
        var balance = stripeClient.checkBalance(payment.getUserId());
        if (balance.compareTo(payment.getAmount()) < 0) {
            return new PaymentResult(PaymentStatus.DECLINED, "Insufficient funds");
        }
        // Process payment...
    }
}
```

#### 4. Verify Tests Pass

```bash
./gradlew test jacocoTestReport
```

#### Frontend Component Testing

1. Unit Testing with Vitest:

```typescript
// PaymentForm.test.tsx
describe('PaymentForm', () => {
  it('should validate card number format', async () => {
    // Given
    const onSubmit = vi.fn();
    render(<PaymentForm onSubmit={onSubmit} />);

    // When
    await userEvent.type(screen.getByLabelText(/card number/i), '4242');
    await userEvent.click(screen.getByRole('button', { name: /submit/i }));

    // Then
    expect(screen.getByText(/invalid card number/i)).toBeInTheDocument();
    expect(onSubmit).not.toHaveBeenCalled();
  });
});
```

2. E2E Testing with Playwright:

```typescript
// payment.spec.ts
test("complete payment flow", async ({ page }) => {
  // Given
  await page.goto("/checkout");
  await page.fill('[data-testid="card-number"]', "4242424242424242");

  // When
  await page.click('button:has-text("Pay Now")');

  // Then
  await expect(page.locator(".payment-success")).toBeVisible();
  await expect(page.locator(".order-id")).toContainText(/\d+/);
});
```

3. Component Mocking Pattern:

```typescript
// PaymentContext.test.tsx
const mockStripeClient = {
  createPayment: vi.fn(),
  getBalance: vi.fn(),
};

vi.mock("../services/stripe", () => ({
  useStripeClient: () => mockStripeClient,
}));

test("handles payment failure gracefully", async () => {
  // Given
  mockStripeClient.createPayment.mockRejectedValue(
    new Error("Insufficient funds"),
  );

  // When/Then
  await expect(submitPayment(100)).rejects.toThrow("Insufficient funds");
  expect(notifyError).toHaveBeenCalled();
});
```

Testing Requirements:

- Backend: JUnit tests required, >85% coverage (check `backend/build/jacocoHtml/index.html`)
- Frontend: Vitest for unit tests, Playwright for E2E
- Each PR must include tests covering new functionality
- Frontend components must include error state testing

### Code Style

- Java: Checkstyle enforced, package root `com.platform.*`
- TypeScript: ESLint + Prettier, PascalCase components
- Python: PEP 8, `src/<module>/`, `tests/<suite>/test_*.py`
- Indentation: 2 spaces (TS/JS), 4 spaces (Java/Python)

### Key Integration Points

1. Authentication: OAuth2 with opaque tokens
2. Real-time: WebSocket for live updates (with SSE fallback)
3. Storage: PostgreSQL (primary), Redis (cache/session)

### Caching Strategy

The platform uses a multi-layer caching approach:

#### Backend Redis Caching

```java
@Service
class PaymentService {
    @Cacheable(value = "payments", key = "#userId")
    public List<Payment> getUserPayments(String userId) {
        return paymentRepository.findByUserId(userId);
    }

    @CacheEvict(value = "payments", key = "#userId")
    public void processNewPayment(String userId, Payment payment) {
        paymentRepository.save(payment);
    }
}
```

#### Frontend Cache Management

```typescript
// RTK Query cache configuration
export const apiSlice = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
  tagTypes: ["Payment", "User", "Subscription"],
  endpoints: (builder) => ({
    getPayments: builder.query({
      query: () => "payments",
      providesTags: ["Payment"],
    }),
    addPayment: builder.mutation({
      query: (payment) => ({
        url: "payments",
        method: "POST",
        body: payment,
      }),
      invalidatesTags: ["Payment"],
    }),
  }),
});
```

Cache Invalidation Rules:

- Use specific cache keys (e.g., `payments:${userId}`)
- Implement TTL for volatile data
- Clear related caches on updates (e.g., payment affects subscription)

### Troubleshooting Guide

#### 1. Development Environment Issues

```bash
# Redis connection issues
docker compose restart redis
docker compose logs redis   # Check for errors

# Database migration failed
./gradlew flywayClean      # Reset migrations
./gradlew flywayMigrate    # Rerun migrations

# Node modules issues
cd frontend
rm -rf node_modules package-lock.json
npm install

# TypeScript type errors
npm run clean              # Clean TypeScript cache
npm run type-check         # Verify types

# Git lock issues
make git-clean-lock       # Remove stale lock files
```

#### 2. Common Runtime Errors

- `ModuleNotFoundException`: Check module boundaries in `backend/src/main/java/com/sass/`
- `WebSocket Connection Failed`: Verify Redis is running for session management
- `Invalid Cache State`: Clear Redis cache with `docker compose exec redis redis-cli FLUSHALL`
- `LazyInitializationException`: Missing `@Transactional` or eager loading needed
- `OptimisticLockException`: Concurrent updates, implement retry mechanism
- `WebpackChunkLoadError`: Clear browser cache and rebuild frontend
- `InvalidTokenError`: Clear browser storage and re-authenticate

#### 3. Testing Issues

- Flaky tests: Use `@RetryTest` annotation for network-dependent tests
- CI failures: Check `.github/workflows/` for environment variables
- E2E timeouts: Increase timeout in `playwright.config.ts`
- Random test failures: Check for test isolation issues and shared state
- Outdated snapshots: Run `npm test -- -u` to update
- Database deadlocks: Add `@Transactional(isolation = READ_COMMITTED)`

#### 4. Performance Issues

- Slow queries: Check `backend/logs/slow-queries.log`
- Memory leaks: Enable heap dumps with `-XX:+HeapDumpOnOutOfMemoryError`
- Frontend rendering: Use React DevTools profiler
- N+1 queries: Enable SQL debug logging and use `@BatchSize`

#### 5. Deployment Issues

- Version mismatch: Check `package.json` and `build.gradle` versions
- Missing env vars: Compare with `.env.example`
- Container startup: Check logs with `docker compose logs -f service-name`
- Database connections: Verify connection pool settings in `application.yml`

### Common Gotchas

- Always check `.env.example` for required environment variables
- Run `make pre-commit` before submitting changes
- Database migrations must be backward compatible
- Frontend components must handle loading/error states
- Clear Redis cache when switching branches with schema changes

### Reference Files

- Architecture: `docs/docs/architecture/overview.md`
- API Contracts: `specs/*/contracts/`
- Component Examples: `frontend/src/components/project/`
- Module Boundaries: `backend/src/main/java/com/sass/*/`

## Constitutional Requirements

1. Library-first development
2. CLI interfaces for tools
3. Test-driven development
4. Real dependencies in tests
5. Structured logging with correlation IDs
6. Semantic versioning

---
> Source: [lsendel/sass](https://github.com/lsendel/sass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
