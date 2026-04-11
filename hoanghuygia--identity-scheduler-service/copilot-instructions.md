## identity-scheduler-service

> This is a Spring Boot 3.3.4 Java 21 auth service (package `com.example.authservice`).

# AGENTS.md

This is a Spring Boot 3.3.4 Java 21 auth service (package `com.example.authservice`).
It uses Maven, JPA/Hibernate, Spring Security (stateless JWT), Flyway, MapStruct, and Lombok.

## Build / Test Commands

```bash
# Full build + tests (what CI runs)
mvn -B clean verify

# Compile only (skip tests)
mvn -B compile

# Run all tests
mvn -B test

# Run a single test class
mvn -B test -Dtest=AuthServiceApplicationTests

# Run a single test method
mvn -B test -Dtest=AuthServiceApplicationTests#contextLoads

# Run tests matching a pattern
mvn -B test -Dtest="com.example.authservice.**.*Test"

# Package (build JAR, skip tests)
mvn -B -DskipTests package

# Build Docker image
docker build -t authservice:ci .
```

Tests use `@ActiveProfiles("test")` which activates `application-test.yml` (H2 in-memory DB,
Flyway disabled). No external database is needed to run tests.

There is no separate linter or formatter configured. No Checkstyle, Spotless, or PMD plugins exist.

## Project Structure

```
src/main/java/com/example/authservice/
  auth/         -- Authentication (controller, service, DTOs, mapper, validator)
  audit/        -- Audit logging (entity, repository, service)
  common/       -- Cross-cutting: exceptions, response wrappers, filters, logging, util
  config/       -- AppProperties, MailConfig, OpenApiConfig
  mail/         -- Email service (stub impl + template builder)
  role/         -- Role entity, repository, service
  security/     -- Spring Security config, JWT filter/service, UserDetailsService
  token/        -- Refresh, password-reset, email-verification tokens
  user/         -- User entity, DTOs, repository, service
```

Packages are **feature-based** with layered sub-packages: `entity/`, `repository/`, `service/`,
`dto/`, `controller/`, `mapper/`, `validator/`.

## Code Style Guidelines

### Imports

- **Order**: Project (`com.example.*`) > Third-party (`jakarta.*`, `lombok.*`, `org.*`) > JDK (`java.*`)
- One blank line between each group. Alphabetical within groups.
- **No wildcard imports** -- always import individual classes.

### Naming Conventions

| Element            | Convention               | Examples                                        |
|--------------------|--------------------------|-------------------------------------------------|
| Packages           | Lowercase singular nouns | `auth`, `user`, `token`, `common`               |
| Entities           | No suffix                | `User`, `Role`, `RefreshToken`                  |
| Repositories       | Suffix `Repository`      | `UserRepository`, `RoleRepository`              |
| Services (iface)   | Suffix `Service`         | `AuthService`, `UserService`                    |
| Services (impl)    | Suffix `ServiceImpl`     | `AuthServiceImpl`, `UserServiceImpl`            |
| Controllers        | Suffix `Controller`      | `AuthController`                                |
| Request DTOs       | Suffix `Request`         | `LoginRequest`, `RegisterRequest`               |
| Response DTOs      | Suffix `Response`/`Dto`  | `AuthResponse`, `UserSummaryDto`                |
| Mappers            | Suffix `Mapper`          | `AuthMapper`                                    |
| Filters            | Suffix `Filter`          | `JwtAuthenticationFilter`                       |
| Config classes     | Suffix `Config`          | `SecurityConfig`, `MailConfig`                  |
| Enums              | UPPER_SNAKE_CASE values  | `LOGIN_SUCCESS`, `VALIDATION_ERROR`             |
| Constants          | UPPER_SNAKE_CASE         | `REQUEST_ID_HEADER`, `TRACE_ID`                 |
| Methods            | camelCase, verb-first    | `register()`, `findByEmail()`, `isTokenValid()` |
| Fields             | camelCase                | `passwordHash`, `emailVerified`, `lastLoginAt`  |

### DTOs

- **Always Java records**, never classes. No `@Data` or `@Value` from Lombok.
- Input DTOs use `jakarta.validation.constraints` annotations (`@NotBlank`, `@Email`, `@Size`, `@Pattern`).
- Output DTOs are plain records with no annotations.
- Static factory methods are acceptable on records (e.g., `AuthResponse.stub()`).

### Entities

- No shared base entity class. Each entity declares its own ID and audit fields.
- IDs are `UUID`, app-generated (no `@GeneratedValue`).
- Timestamps use `java.time.Instant` with `@CreationTimestamp` / `@UpdateTimestamp` (Hibernate).
- All `@ManyToOne` use `fetch = FetchType.LAZY` explicitly.
- All `@OneToMany` use `mappedBy` (bidirectional) and `fetch = FetchType.LAZY`.
- Collections initialized to `new HashSet<>()`.
- Enums persisted with `@Enumerated(EnumType.STRING)`.
- Table names are plural snake_case: `users`, `refresh_tokens`, `auth_audit_logs`.
- Column names are snake_case in `@Column(name = "...")` with explicit constraints.

### Lombok Usage

- **`@Getter` + `@Setter`** on entities and config classes (never `@Data`).
- **`@Getter` + `@Builder`** on response wrappers (`ApiResponse`, `ErrorResponse`).
- **`@RequiredArgsConstructor`** for constructor injection -- all injected fields must be `final`.
- **`@Slf4j`** for logging (never manual `LoggerFactory.getLogger()`).
- **`@NoArgsConstructor` + `@AllArgsConstructor` + `@EqualsAndHashCode`** on `@Embeddable` composite keys.

### Annotation Order on Classes

```java
// Controllers:          @RestController > @RequestMapping > @RequiredArgsConstructor
// Service impls:        @Slf4j (if logging) > @Service > @RequiredArgsConstructor
// Entities:             @Getter > @Setter > @Entity > @Table
// Config:               @Configuration > @Enable* > @RequiredArgsConstructor
// Filters:              @Slf4j > @Component > @Order
```

### Dependency Injection

- Always constructor injection via Lombok `@RequiredArgsConstructor`.
- All injected dependencies are `private final` fields.
- No field injection (`@Autowired` on fields) or setter injection.

### Service Layer

- Every service has an **interface** (no annotations) and a separate **`*Impl`** class (`@Service`).
- Repositories extend `JpaRepository<Entity, UUID>` with Spring Data query methods.

### API Response Pattern

- **Success**: Return `ResponseEntity<ApiResponse<T>>` using `ApiResponse.success(message, data)`.
- **Errors**: Handled exclusively by `GlobalExceptionHandler`, returning `ErrorResponse`.
- Controllers never build error responses manually.
- Throw `AppException(ErrorCode, HttpStatus, message)` for business errors.

### Error Handling

- Single exception class: `AppException extends RuntimeException` with `ErrorCode` enum and `HttpStatus`.
- `@RestControllerAdvice` global handler catches (in order):
  1. `MethodArgumentNotValidException` -> 400
  2. `ConstraintViolationException` -> 400
  3. `AppException` -> status from exception
  4. `Exception` (catch-all) -> 500

### Logging

- Use `@Slf4j` annotation, then `log.info(...)`, `log.error(...)`, etc.
- **Structured key=value format**: `log.info("event_name key1={} key2={}", val1, val2)`
- Use SLF4J parameterized messages -- never string concatenation.
- Request/trace IDs are in MDC via `RequestTracingFilter` and available through `RequestContextUtil`.

### MapStruct Mappers

- Defined as Java interfaces with `@Mapper(componentModel = "spring")`.
- Every non-mappable target field must have an explicit `@Mapping(target = "...", ignore = true)`.
- Default values via `@Mapping(target = "...", constant = "...")`.
- Method naming: `toEntity(dto)` pattern.

### Security

- Stateless JWT with `SecurityFilterChain` bean in `SecurityConfig`.
- Method-level security via `@PreAuthorize("hasRole('...')")`.
- `BCryptPasswordEncoder` for password hashing.
- `JwtTokenService` interface abstracts JWT operations.

### Database Migrations

- Flyway migrations in `src/main/resources/db/migration/`.
- Naming: `V{number}__{description}.sql` (e.g., `V1__init_auth_schema.sql`).
- Flyway is disabled in the test profile (H2 is used instead).

### Spring Profiles

| Profile | Database   | Flyway  | Use case          |
|---------|------------|---------|--------------------|
| `dev`   | PostgreSQL | Enabled | Local development  |
| `test`  | H2         | Disabled| Tests              |
| `prod`  | PostgreSQL | Enabled | Production         |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hoanghuygia)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Hoanghuygia)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
