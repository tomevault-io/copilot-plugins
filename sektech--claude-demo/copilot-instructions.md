## claude-demo

> A RESTful CRUD API for managing a product catalog. Single entity (`Product`), fully documented with Swagger UI, profile-based datasource configuration, and full-stack integration tests.

# spring-crud-api — Project Knowledge

## Overview

A RESTful CRUD API for managing a product catalog. Single entity (`Product`), fully documented with Swagger UI, profile-based datasource configuration, and full-stack integration tests.

---

## Tech Stack

| Component       | Technology                                      |
|-----------------|-------------------------------------------------|
| Language        | Java 21                                         |
| Framework       | Spring Boot 3.5.2                               |
| Web             | Spring MVC (`spring-boot-starter-web`)          |
| ORM             | Spring Data JPA + Hibernate 6.6                 |
| Validation      | Jakarta Bean Validation (`spring-boot-starter-validation`) |
| Dev DB          | H2 in-memory                                    |
| Prod DB         | Oracle (ojdbc11) via `devint` profile           |
| API Docs        | SpringDoc OpenAPI 2.8.8 (Swagger UI)            |
| Build           | Gradle 8.12.1 (Gradle Wrapper)                  |
| Testing         | JUnit 5 + Spring Boot Test + MockMvc            |
| Coverage        | JaCoCo — 80% minimum enforced                   |
| Hot Reload      | Spring DevTools                                 |

---

## Architecture

3-layer (n-tier) architecture — standard Spring Boot web application pattern:

```
HTTP Request
     │
     ▼
┌─────────────────────┐
│  ProductController  │  ← Layer 1: Web / API layer
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   ProductService    │  ← Layer 2: Business logic layer
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  ProductRepository  │  ← Layer 3: Data access layer
└─────────┬───────────┘
          │
          ▼
     Database (H2 / Oracle)
```

Supporting classes:
- `GlobalExceptionHandler` — `@RestControllerAdvice`, intercepts exceptions across all layers
- `ResourceNotFoundException` — custom `RuntimeException` mapped to HTTP 404
- `OpenApiConfig` — Swagger/OpenAPI metadata bean
- `CrudApplication` — `@SpringBootApplication` entry point

---

## Package Structure

```
src/main/java/com/example/crud/
├── CrudApplication.java
├── config/
│   └── OpenApiConfig.java
├── controller/
│   └── ProductController.java
├── exception/
│   ├── GlobalExceptionHandler.java
│   └── ResourceNotFoundException.java
├── model/
│   └── Product.java
├── repository/
│   └── ProductRepository.java
└── service/
    └── ProductService.java

src/main/resources/
├── application.properties          ← common config, activates 'local' profile
├── application-local.properties    ← H2 in-memory, H2 console enabled
├── application-devint.properties   ← Oracle DB, H2 console disabled
└── data.sql                        ← seed data (3 products on startup)

src/test/java/com/example/crud/
└── ProductControllerTest.java
```

---

## API Endpoints

Base path: `/api/products`

| Method | Path           | Description           | Success Status |
|--------|----------------|-----------------------|----------------|
| GET    | `/`            | Get all products      | 200            |
| GET    | `/{id}`        | Get product by ID     | 200 / 404      |
| POST   | `/`            | Create product        | 201            |
| PUT    | `/{id}`        | Update product        | 200 / 404      |
| DELETE | `/{id}`        | Delete product        | 204 / 404      |

Swagger UI: `http://localhost:8080/swagger-ui.html`
OpenAPI JSON: `http://localhost:8080/api-docs`

---

## Product Entity

```java
@Entity @Table(name = "products")
public class Product {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank  private String name;
               private String description;
    @NotNull @Positive private Double price;
    @NotNull @Positive private Integer quantity;
}
```

Validation is enforced at the controller boundary via `@Valid`. Errors are caught by `GlobalExceptionHandler` and returned as structured JSON with field-level messages.

---

## Coding Style & Conventions

**Constructor injection** — no `@Autowired` on fields. All dependencies are `final` and injected via constructor. Applied consistently in `ProductController` and `ProductService`.

**Thin controller, logic in service** — the controller only maps HTTP verbs and wraps responses in `ResponseEntity`. All business decisions (including the existence check before delete) live in the service layer.

**Exception as flow control** — `ProductService.getById()` throws `ResourceNotFoundException` when not found. `update()` and `delete()` reuse this by calling `getById()` first — no duplicated existence checks.

**Profile-based config** — three property files cover local dev (H2), integration (Oracle), and shared common settings. Default active profile is `local` (set in `application.properties`).

**Seed data** — `data.sql` inserts 3 products on startup. Works because `ddl-auto=create-drop` creates the schema first, then Spring runs the SQL file automatically.

**No DTOs** — the `Product` JPA entity is used directly as the API request/response body. Acceptable for a simple CRUD API; would need a DTO layer if the entity grows divergent from the API contract.

**No comments in code** — names are self-describing. Comments would only be added for non-obvious invariants or workarounds.

---

## Exception Handling

`GlobalExceptionHandler` (`@RestControllerAdvice`) handles two cases:

| Exception                        | HTTP Status | Response body                          |
|----------------------------------|-------------|----------------------------------------|
| `ResourceNotFoundException`      | 404         | `{ timestamp, status, error }`         |
| `MethodArgumentNotValidException`| 400         | `{ timestamp, status, errors: { field: message } }` |

---

## Configuration Profiles

| Profile   | Database           | DDL auto    | H2 Console |
|-----------|--------------------|-------------|------------|
| `local`   | H2 in-memory       | create-drop | enabled    |
| `devint`  | Oracle (port 1521) | update      | disabled   |

Switch profile: `-Dspring.profiles.active=devint`

---

## Build & Run

```powershell
# Run tests + generate coverage report
.\gradlew.bat test

# Run the application (uses JDK 21 toolchain automatically)
.\gradlew.bat bootRun

# Build JAR
.\gradlew.bat build
```

**Important:** Always use `.\gradlew.bat bootRun` to run the app — do NOT run the JAR directly with `java -jar` unless `JAVA_HOME` points to JDK 21. The Gradle toolchain compiles bytecode to class file version 65.0 (Java 21); running with JDK 17 throws `UnsupportedClassVersionError`.

---

## JUnit Tests

**File:** `src/test/java/com/example/crud/ProductControllerTest.java`

**Strategy: Full-stack integration tests** — `@SpringBootTest` + `@AutoConfigureMockMvc` boots the complete Spring context with a real H2 database. Requests flow through MockMvc → Controller → Service → Repository → H2. No mocking of internal layers.

**Isolation:** `@BeforeEach` calls `repository.deleteAll()` — every test starts with an empty database and creates only the data it needs.

| Test                | Covers                                          |
|---------------------|-------------------------------------------------|
| `createProduct()`   | POST → 201, response has `id` and correct name  |
| `getAll()`          | GET → 200, correct list length                  |
| `getById_notFound()`| GET with unknown ID → 404                       |
| `updateProduct()`   | PUT → 200, updated field value in response      |
| `deleteProduct()`   | DELETE → 204                                    |

**Known gap:** `GET /api/products/{id}` happy path (200) has no test — `ProductController.getById()` is the one uncovered method in JaCoCo. All service and model code is at 100%.

---

## JaCoCo Coverage

Configured in `build.gradle`. Minimum threshold: **80%** (enforced via `jacocoTestCoverageVerification`).

Excluded from coverage measurement:
- `CrudApplication` (entry point, no logic)
- `config/**` (OpenApiConfig — pure wiring)
- `exception/**` (thin exception classes)

Last measured coverage (post Java 21 upgrade):

| Class              | Instructions | Lines | Methods |
|--------------------|-------------|-------|---------|
| `ProductService`   | 100%        | 100%  | 100%    |
| `ProductController`| 84%         | 89%   | 83%     |
| `Product`          | 100%        | 100%  | 100%    |
| **Overall**        | **96%**     | **98%**| **96%**|

---

## Known Warnings

**Mockito dynamic agent (JDK 21):** Mockito self-attaches at test runtime and prints a warning about dynamic agent loading being disallowed in future JDK releases. Suppress with:

```groovy
tasks.named('test') {
    jvmArgs '-XX:+EnableDynamicAgentLoading'
}
```

**H2Dialect deprecation:** Hibernate logs `HHH90000025: H2Dialect does not need to be specified explicitly`. Remove `spring.jpa.database-platform=org.hibernate.dialect.H2Dialect` from `application-local.properties` to silence it.

---
> Source: [sektech/claude-demo](https://github.com/sektech/claude-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
