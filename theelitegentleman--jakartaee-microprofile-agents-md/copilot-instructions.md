## jakartaee-microprofile-agents-md

> Jakarta EE + MicroProfile application (Java 17 or 21 LTS).

# AGENTS.md

## Project overview
Jakarta EE + MicroProfile application (Java 17 or 21 LTS).  
Uses CDI, JAX-RS, JPA/Hibernate (or EclipseLink), JSON-B, and core MicroProfile specs (Config, Health, Metrics, OpenAPI, REST Client, JWT Auth, Fault Tolerance, etc.).  
Runtime is typically Quarkus, Open Liberty, WildFly, Payara, or Helidon — check `pom.xml` / `build.gradle` and runtime-specific configuration files.

## Build system
- Prefer the Maven Wrapper (`./mvnw`) over a global `mvn`.
- Java version: match the one declared in the POM/toolchain (usually 17 or 21).
- Never introduce Gradle unless the project already uses it.

### Common Maven commands
```bash
./mvnw clean package                    # full build (runs tests)
./mvnw clean package -DskipTests        # fast iteration
./mvnw clean install -T 1C              # parallel build when safe
./mvnw test                             # unit + integration tests
./mvnw -Dtest=MyResourceTest test       # single test class
./mvnw quarkus:dev                      # Quarkus live coding (if applicable)
./mvnw liberty:dev                      # Open Liberty dev mode (if applicable)
./mvnw wildfly:run                      # WildFly (if applicable)
```

## Project structure
- `src/main/java` — application code (resources, services, repositories, entities, config)
- `src/main/resources` — `application.properties` / `microprofile-config.properties`, `persistence.xml`, `beans.xml`, OpenAPI, etc.
- `src/test/java` — tests mirroring the main package structure
- `src/test/resources` — test configuration and Arquillian / Quarkus test resources
- `pom.xml` — single source of truth for dependencies and plugins

## Core conventions
- Prefer **CDI** (`@ApplicationScoped`, `@RequestScoped`, `@Inject`, `@Produces`) over other DI mechanisms.
- Use constructor or field injection with CDI; avoid manual `new` for managed beans.
- JAX-RS resources should be thin; put business logic in CDI services.
- Prefer **MicroProfile Config** (`@ConfigProperty`) over hardcoded values or direct `System.getenv`.
- Use **MicroProfile Rest Client** for outgoing HTTP calls when available.
- Prefer **JSON-B** (or the runtime’s default JSON provider) for serialization.
- Entities belong in the persistence layer; never expose JPA entities directly from JAX-RS endpoints — use DTOs or records.
- Keep beans stateless where possible. Prefer immutable data carriers (records) for DTOs and value objects.
- Use the project’s existing logging approach (usually `java.util.logging`, JBoss Logging, or SLF4J via the runtime).
- Avoid methods using a `throws` clause; prefer unchecked exceptions.

## Persistence & Repository pattern
- If **Jakarta Data** is **not** used in the project, apply the classic **Repository pattern** for all data/object persistence.
  - Define repository interfaces (e.g. `UserRepository`) that abstract persistence operations.
  - Provide CDI-managed implementations that use `EntityManager`, Panache, or the project’s existing persistence technology.
  - Services must depend on the repository abstraction, never on `EntityManager` directly (unless the project already does so consistently).
- Keep repositories focused solely on data access; business rules belong in services.

## Exception handling
- Business and repository exceptions **must be descriptive** of the exact cause.
- Prefer specific, domain-meaningful runtime exceptions over generic ones.
- Example: when a user cannot be found, throw a dedicated `UserNotFoundException` (extending `RuntimeException` or the project’s base business exception) instead of a generic `NotFoundException` or `RuntimeException`.
- Include relevant context in the exception message (e.g. the ID or username that was not found) when it aids debugging, without leaking sensitive data.
- Do not swallow exceptions; let descriptive exceptions propagate or map them cleanly at the JAX-RS boundary (via `ExceptionMapper` if the project uses them).

## Code style
- Follow the project’s formatter (Spotless, Checkstyle, or IDE defaults). Run formatting before committing.
- No star imports.
- Use `@Override` annotation when overriding methods.
- Prefer explicit types; use `var` only when the type is obvious.
- Public APIs should have clear Javadoc.
- Prefer constructor injection for required dependencies.
- Use `@Transactional` (Jakarta Transactions) on service methods that need it; do not open EntityManagers manually unless required.

## Testing
- Primary framework: JUnit 5.
- Prefer runtime-native testing:
  - Quarkus → `@QuarkusTest` + RestAssured
  - Open Liberty/others → Arquillian or runtime-specific test support when present
- Use RestAssured for JAX-RS endpoint tests.
- Add or update tests for every behavioural change. Never weaken or delete existing tests to make a change pass.
- Prefer unit tests for pure logic; integration/container tests only when necessary.
- Use Testcontainers or the runtime’s Dev Services when the project already does so.

## Architecture & layering rules
- Typical layers:  
  `JAX-RS Resource` → `CDI Service` → `Repository / EntityManager` / external clients
- Resources must not contain business logic or direct persistence access.
- Services orchestrate; repositories (or EntityManager) handle persistence.
- Do not introduce new third-party libraries unless clearly justified and already used elsewhere in the project.
- Respect existing package structure and naming conventions.

## MicroProfile & Jakarta EE specifics
- Health: implement `HealthCheck` for readiness/liveness when the project uses MicroProfile Health.
- Metrics: use `@Timed`, `@Counted`, etc., consistently with existing code.
- OpenAPI: keep annotations (`@Operation`, `@APIResponse`, etc.) accurate when present.
- Fault Tolerance: use `@Retry`, `@Timeout`, `@CircuitBreaker`, `@Fallback` where the project already applies them.
- JWT: follow the existing MicroProfile JWT Auth configuration and role checks.
- Config: externalise all environment-specific values via MicroProfile Config; never hard-code URLs, credentials, or ports.

## Security constraints
- Never commit secrets, private keys, or real credentials.
- Validate and sanitise all external input.
- Use the container’s security mechanisms (or MicroProfile JWT) rather than rolling your own authentication.
- Prefer HTTPS and secure defaults already configured in the project.

## Pull requests & commits
- Run the full verification command before committing:
  ```bash
  ./mvnw clean verify
  ```
  (or the project’s equivalent quality gate).
- Keep commits focused and messages clear.
- Ensure new code follows the layering, CDI, and DTO rules above.

## Notes for agents
- Always check the actual runtime (Quarkus, Open Liberty, WildFly, etc.) and the `pom.xml` before assuming commands or annotations.
- Prefer the most specific existing pattern in neighbouring classes over inventing new ones.
- When adding a new MicroProfile or Jakarta EE feature, look for existing usage in the codebase first.

---
> Source: [TheEliteGentleman/jakartaee-microprofile-agents-md](https://github.com/TheEliteGentleman/jakartaee-microprofile-agents-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
