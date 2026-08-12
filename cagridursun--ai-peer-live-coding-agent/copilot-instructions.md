## spring-boot-patterns

> When building a new service, look for proven patterns in your own practice projects or open-source Spring Boot examples:

# Spring Boot — Preferred Patterns & Conventions

## Reference Code
When building a new service, look for proven patterns in your own practice projects or open-source Spring Boot examples:
- [EXAMPLE_PROJECT_PATH/URL] conversational / workflow APIs (state machines, step validation)
- [EXAMPLE_PROJECT_PATH/URL] CRUD REST APIs (standard controller → service → repository)
- [EXAMPLE_PROJECT_PATH/URL] OpenAPI-first services (codegen for interfaces and DTOs)

Consult those references for project structure, Maven config, test patterns, and error handling — adapt to the interview scope, not copy wholesale.

## Architecture
- 3-layer max: Controller → Service → Repository
- Kotlin for entity/model classes only (no Lombok); Java for all other code — optional, use when the problem fits
- OpenAPI codegen for controller interfaces and DTOs (`interfaceOnly=true`) — optional
- JPA + H2 in-memory with `create-drop` DDL — when persistence is required
- `@RestControllerAdvice` for centralized error responses
- `BigDecimal` for monetary values
- `ConcurrentHashMap` for in-memory rate limiting or caches

## Maven / Build
- Kotlin compiler plugins: `spring` + `jpa` (allopen + noarg) — when using Kotlin entities
- `kotlin-maven-noarg` dependency required for JPA entity no-arg constructors
- Kotlin compiles before Java; OpenAPI generated sources in Kotlin `sourceDirs`
- Java `default-compile` phase `none`, custom `java-compile` after Kotlin — when mixing Kotlin + Java
- Set `maven.compiler.source`, `maven.compiler.target`, `maven.compiler.release` explicitly in `<properties>` for IDE compatibility
- JaCoCo 0.8.14+ for Java 21+ (older versions may fail on newer class file versions)
- Exclude generated API packages from JaCoCo coverage reports

## Error Handling
- Custom exceptions: NotFoundException (404), ConflictException (409), ValidationException (400), RateLimitException (429)
- `GlobalExceptionHandler` maps to `ErrorResponse` with `code`, `message`, `details` (list of `FieldError`)
- Filter-thrown exceptions: inject `HandlerExceptionResolver`, call `resolveException()` to route through `@RestControllerAdvice`
- Always structured JSON, never plain text

## Testing
- Integration: `@SpringBootTest` + `@AutoConfigureMockMvc`, extract IDs with `JsonPath.read()`
- Unit: `@ExtendWith(MockitoExtension.class)` + `@Mock` + `@InjectMocks`
- No global `@BeforeEach` stubs that not all tests use (strict stubbing rejects them)
- Disable filters in tests via config property (e.g. `rate-limit.enabled=false` in test `application.yml`)
- Separate H2 URL in test config when using JPA

## Common Gotchas
- PowerShell: no `&&` operator, use `;` or separate commands
- Switch expressions need Java 14+; set compiler properties explicitly
- JPA entities in Kotlin need `class` not `data class` for mutable `var` fields
- `@Embedded` nullable objects need all columns nullable or use `@Column` defaults
- `@ElementCollection` needs `@CollectionTable` + `@OrderColumn` for ordered lists

## Execution Priority
1. Boot + health check
2. One endpoint end-to-end + test
3. Next endpoint + test
4. Validation + error handling + tests
5. Edge cases
6. Docker / extras only if time allows

---
> Source: [cagridursun/ai-peer-live-coding-agent](https://github.com/cagridursun/ai-peer-live-coding-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
