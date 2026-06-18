## dbchatbox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Common Commands

```bash
# Build (skip tests)
./gradlew build -x test

# Build with tests
./gradlew build

# Run application (dev profile, H2 in-memory DB)
./gradlew bootRun

# Run all tests
./gradlew test

# Run a single test class
./gradlew test --tests ClassName

# Run a single test method
./gradlew test --tests ClassName.methodName

# Mutation testing
./gradlew pitest
```

## Architecture

This is a **GraphQL-only** club management system built with DDD layering. There are no REST endpoints.

```
src/main/java/at/mavila/dbchatbox/
├── domain/
│   ├── club/
│   │   ├── exception/      # Domain exceptions (MemberNotFound, DuplicateEmail, InvalidStatusTransition, ...)
│   │   ├── member/         # Member + status history + GDPR anonymization
│   │   ├── membership/     # MembershipType + grace period
│   │   ├── notification/   # NotificationService interface (logging-only mock in Phase 1)
│   │   ├── payment/        # Payment + PaymentDocument upload/review workflow
│   │   ├── subscription/   # MemberSubscription + SubscriptionPaymentStatus
│   │   ├── trainer/        # Trainer + TrainerSettings + TrainerLog
│   │   └── training/       # Session + SessionOccurrence
│   ├── chatbox/            # Natural-language assistant (Spring AI)
│   │   ├── exception/      # ChatRateLimitExceeded, ChatProviderUnavailable
│   │   └── tools/          # @Tool-annotated read wrappers over domain services
│   └── support/            # TSID generator, CommandValidator (Jakarta Bean Validation helper)
└── infrastructure/
    ├── ai/                 # ChatClient configuration + system prompt + Clock bean
    ├── web/graphql/        # Per-entity controllers + ChatAssistantController, GraphQlExceptionAdvice, ScalarConfiguration
    └── scheduling/         # GdprPurgeJob (daily 02:00)
```

**Request flow:** GraphQL controller (`@QueryMapping` / `@MutationMapping` / `@SchemaMapping`) → domain service → JPA repository. There is no intermediate application-service layer — controllers depend on domain services directly.

**Key files:**
- `src/main/resources/graphql/schema.graphqls` — all queries, mutations, scalars, and types
- `src/main/resources/application.properties` — GDPR cron (`app.gdpr.purge-cron`, `app.gdpr.retention-days=30`), payment/notification settings
- `src/main/resources/application-{dev,test,prod}.properties` — profile-specific config
- `src/main/resources/db/migration/V1..V5_*.sql` — Flyway versioned migrations (`ddl-auto=validate`, so schema changes require a new migration)
- `src/specs/club/ClubManagement.md` — product spec driving the feature set

**Tech stack:** Java 25 · Spring Boot 4.0.5 · Spring for GraphQL · JPA/Hibernate 7 · Flyway · H2 (dev/test) · PostgreSQL 16 (prod) · Lombok · TSID IDs · graphql-java-extended-scalars · Jakarta Bean Validation · Spring AI 2.0.0-M3 (Anthropic) for the chatbox · PIT mutation testing

All entities use JPA `@Version` (stored as `Short`) for optimistic locking.

## Adding a New Feature (Spec-Driven Pipeline)

Specs live in `src/specs/<category>/`. When implementing a spec, touch these layers in order:

1. **Domain** — `@Service`/`@Component` in `domain/club/<category>/`, command `record` with Jakarta Bean Validation annotations, custom exception in `domain/club/exception/` if needed. If the schema changes, add a Flyway migration (next `V{n}__*.sql`).
2. **Infrastructure** — add type/query/mutation to `schema.graphqls`, add `@QueryMapping`/`@MutationMapping` to the matching per-entity controller (or create a new `XController`), add handler to `GraphQlExceptionAdvice` if the exception needs a dedicated GraphQL error mapping.
3. **Tests** — domain test (`@SpringBootTest`, AssertJ), GraphQL integration test via `ExecutionGraphQlServiceTester` with raw query documents.
4. **Docs** — Javadoc on every new/modified class and public method, update `README.md` if user-visible behavior changes.

## Coding Conventions

These are enforced by static analysis (CodeScene); violations are flagged as `Complex Method`, `Complex Conditional`, `Code Duplication`, `Excess Number of Function Arguments`.

### Null checks
Use static imports — never `== null` / `!= null`:
```java
import static java.util.Objects.isNull;
import static java.util.Objects.nonNull;

if (isNull(input)) { ... }
```

### Method parameters
Public methods must have **≤ 3 parameters**. Group more into a command/parameter `record`:
```java
public record CreateMemberCommand(String firstName, String lastName, String email, ...) {}
public Member createMember(final CreateMemberCommand command) { ... }
```

### Input validation
Domain services must **not** contain manual validation guards. Use Jakarta Bean Validation on parameter records:
```java
public record MyParams(
    @NotNull(message = "X must not be null")
    @Positive(message = "X must be positive")
    Integer x
) {}
```
Each annotation on its own line, directly above the parameter it annotates.

### Guard clauses
Check the positive/happy-path condition and return early:
```java
// ✅
if (violations.isEmpty()) { return; }
throw violationMapper.toDomainException(violations);

// ❌ — negated condition wrapping the throw
if (!violations.isEmpty()) { throw ...; }
```

### Complex conditionals
Extract compound boolean expressions into named private predicate methods:
```java
// ✅
if (isScheduled(occurrence) && isWithinDateRange(occurrence, from, to)) { ... }

// ❌
if (occurrence.getStatus() == SCHEDULED && !occ.getDate().isBefore(from) && ...) { ... }
```

### One type per file
Every top-level class, enum, record, interface, or annotation in its own `.java` file. No inner types except `private` helpers used exclusively by the enclosing class.

### Other conventions
- Mark all method parameters and unre-assigned local variables `final`
- Prefer Stream API over imperative loops; use enhanced for-each when carrying mutable state
- Prefer `List.of(...)` / `Map.of(...)` for immutable literals
- String messages: use `"...".formatted(...)` or `String.format(...)` — no concatenation with `+`
- Use `var` only when the type is obvious from the right-hand side
- Target cyclomatic complexity ≤ 10 per method; decompose into private helpers
- Use text blocks (`"""..."""`) for multi-line strings in tests

## Testing Patterns

- Domain tests: `@SpringBootTest` (full context, no mocks), `@Autowired` the component under test, AssertJ assertions
- Unit tests that need mocking: `@Mock` + `@InjectMocks` via MockitoExtension
- GraphQL integration tests: `ExecutionGraphQlServiceTester` with raw GraphQL query documents
- Test method naming: `test<Scenario>_<Condition>`
- Cover: happy paths, edge cases, boundary values, constraint violations, null/empty inputs

## Development Profiles

- **dev** — H2 in-memory (PostgreSQL-compatible mode), H2 console at `/h2-console`
- **prod** — PostgreSQL 16

---
> Source: [totopoloco/dbchatbox](https://github.com/totopoloco/dbchatbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
