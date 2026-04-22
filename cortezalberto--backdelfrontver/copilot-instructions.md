## backdelfrontver

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Educational Spring Boot 3.5.6 REST API for user management with comprehensive JavaDoc documentation in Spanish. Demonstrates modern Java development practices including three-layer architecture, Bean Validation, transactional operations, and OpenAPI integration. Uses Java 17 with Gradle build system and H2 in-memory database.

**Target Audience:** Computer science students learning Spring Boot architecture and best practices.

## Build & Run Commands

**Note:** On Windows, use `gradlew.bat` instead of `./gradlew` for all commands below.

**Run the application (production mode):**
```bash
./gradlew bootRun
# Windows: gradlew.bat bootRun
```

**Run with development profile (enables SQL logs and H2 console):**
```bash
./gradlew bootRun --args='--spring.profiles.active=dev'
# Windows: gradlew.bat bootRun --args="--spring.profiles.active=dev"
```

Application runs on `http://localhost:8080`

**Build the project:**
```bash
./gradlew build
```

**Build executable JAR:**
```bash
./gradlew bootJar
```
JAR location: `build/libs/usuarios-api.jar`

**Run JAR:**
```bash
java -jar build/libs/usuarios-api.jar
```

**Run JAR with dev profile:**
```bash
java -jar build/libs/usuarios-api.jar --spring.profiles.active=dev
```

**Run tests:**
```bash
./gradlew test
```

**Clean build:**
```bash
./gradlew clean build
```

## Architecture

**Three-Layer Architecture:**
- **Controller Layer** (`controller/`): REST endpoints at `/api/usuarios` using `ResponseEntity<>` for explicit HTTP status control (201 CREATED, 200 OK, 204 NO CONTENT)
- **Service Layer** (`service/`): Business logic with `@Transactional` on ALL write operations (crearUsuario, actualizarUsuario, eliminarUsuario). Validates entity existence before updates/deletes to prevent race conditions
- **Repository Layer** (`repository/`): Data access using Spring Data JPA

**DTO Pattern:**
- `UsuarioRequestDTO`: Input validation with Jakarta Bean Validation annotations (@NotBlank, @Email, @Size). Used for both create and update operations.
- `UsuarioResponseDTO`: Output mapping, excludes sensitive fields (password). Static factory method `fromEntity()` converts entities to DTOs.
- DTOs separate internal entity representation from API contracts

**Global Exception Handling:**
`GlobalExceptionHandler` uses `@ControllerAdvice` to centralize error handling with standardized `ErrorDTO` responses:
- `MethodArgumentNotValidException`: Bean validation errors (HTTP 400)
- `IllegalArgumentException`: Business logic validation errors like missing users (HTTP 400)
- `ConstraintViolationException`: Database constraint violations (HTTP 400)
- `DataIntegrityViolationException`: Duplicate email detected via `getRootCause()` analysis (HTTP 409 CONFLICT)
- Generic exceptions return HTTP 500 with sanitized messages (no stack trace exposure in production)

**Database Configuration:**
H2 in-memory database configured in `application.properties`:
- URL: `jdbc:h2:mem:usuariosdb`
- H2 Console: `http://localhost:8080/h2-console` (disabled by default, enable with `H2_CONSOLE_ENABLED=true`)
- JPA auto-creates schema on startup (`spring.jpa.hibernate.ddl-auto=update`)
- SQL logging disabled by default (enable with `JPA_SHOW_SQL=true` or use dev profile)
- Entity constraints: All fields are `nullable=false`, email has `unique=true`, lengths defined (nombre: 100, email/password: 255)

## API Documentation

OpenAPI/Swagger UI configured via SpringDoc:
- Access at: `http://localhost:8080/swagger-ui.html`
- Configuration in `config/OpenApiConfig.java`

## API Endpoints

Base path: `/api/usuarios`

- `POST /api/usuarios` - Create user (returns **201 CREATED**, validates input, returns 409 if email exists)
- `GET /api/usuarios` - List all users (returns **200 OK**)
- `GET /api/usuarios/{id}` - Get user by ID (returns **200 OK** or 400 if not found)
- `PUT /api/usuarios/{id}` - Update user (returns **200 OK**, validates input, returns 400 if not found, 409 if email conflicts)
- `DELETE /api/usuarios/{id}` - Delete user (returns **204 NO CONTENT** or 400 if not found)

## JavaDoc Documentation

All source files contain **extensive educational JavaDoc in Spanish** explaining:
- Architecture concepts (three-layer pattern, dependency injection)
- Design patterns (Builder, Factory Method, DTO)
- Spring annotations (@Service, @Transactional, @Valid, @ControllerAdvice)
- Transaction management and race condition prevention
- Optional API and Streams usage
- HTTP status code semantics

**Generate HTML documentation:**
```bash
./gradlew javadoc
```
Output: `build/docs/javadoc/index.html`

**JavaDoc Configuration:** Configured in `build.gradle` with `Xdoclint:none` to allow educational formatting (tables, code examples, multiple heading levels).

## Key Implementation Details

### Transactional Boundaries & Race Condition Prevention

ALL write operations use `@Transactional` to ensure atomicity:
- `crearUsuario()`: Guarantees atomic construction, persistence, and DTO conversion
- `actualizarUsuario()`: Ensures findById + modify + save execute as atomic unit
- `eliminarUsuario()`: Prevents TOCTOU race conditions

The `eliminarUsuario()` method demonstrates the TOCTOU (Time-Of-Check-Time-Of-Use) prevention pattern:

```java
@Transactional
public void eliminarUsuario(Long id) {
    // CORRECT: Uses findById() + delete() with same entity instance
    Usuario usuario = repository.findById(id)
        .orElseThrow(() -> new IllegalArgumentException("No existe usuario"));
    repository.delete(usuario);
}

// INCORRECT pattern (race condition):
// if (repository.existsById(id)) {
//     repository.deleteById(id);  // Another thread could delete between check and delete
// }
```

### Data Flow Pattern

```
HTTP JSON Request
    ↓
UsuarioRequestDTO (validated with @Valid)
    ↓
Usuario entity (built with @Builder)
    ↓
Database persistence
    ↓
Usuario entity (fetched)
    ↓
UsuarioResponseDTO (password excluded)
    ↓
HTTP JSON Response
```

**Key Security Decision:** `UsuarioResponseDTO` intentionally excludes the password field to prevent sensitive data exposure in API responses.

### Lombok Usage

- **@Data**: Generates getters/setters/toString/equals/hashCode for `Usuario` entity
- **@Builder**: Enables fluent object construction: `Usuario.builder().nombre("...").email("...").build()`
- **@RequiredArgsConstructor**: Auto-generates constructor injection for `final` fields in Service/Controller
- **@AllArgsConstructor / @NoArgsConstructor**: Required by JPA for entity instantiation

### Java Records (Java 14+)

DTOs use records for immutability and automatic generation of constructor/getters/equals/hashCode:
```java
public record UsuarioRequestDTO(
    @NotBlank @Size(min=2, max=100) String nombre,
    @NotBlank @Email String email,
    @NotBlank @Size(min=6) String password
) {}
```

### Streams & Functional Programming

`listarUsuarios()` uses Streams API with method reference:
```java
return repository.findAll()
    .stream()
    .map(UsuarioResponseDTO::fromEntity)  // Method reference = usuario -> UsuarioResponseDTO.fromEntity(usuario)
    .toList();
```

### Factory Method Pattern

`UsuarioResponseDTO.fromEntity(Usuario)` centralizes Entity→DTO conversion logic as a static factory method.

## Code Conventions

- Use Lombok annotations (@Data, @Builder, @RequiredArgsConstructor) to reduce boilerplate
- Record types for immutable DTOs
- JPA entities use @Entity, @Table, @Id with explicit @Column constraints (nullable, length, unique)
- Repository interfaces extend JpaRepository for automatic CRUD operations
- Service layer uses @Transactional for write operations and validates entity existence before updates/deletes
- Controllers return ResponseEntity<> for explicit HTTP status code control
- Exception handlers sanitize error messages to prevent information leakage

## Important URLs

- **API Base:** http://localhost:8080/api/usuarios
- **Swagger UI:** http://localhost:8080/swagger-ui.html
- **H2 Console (dev mode only):** http://localhost:8080/h2-console
  - JDBC URL: `jdbc:h2:mem:usuariosdb`
  - User: `sa`
  - Password: (empty)

## Environment Variables

Override configuration defaults:
- `JPA_SHOW_SQL=true`: Enable SQL query logging
- `H2_CONSOLE_ENABLED=true`: Enable H2 web console

Or use the `dev` profile: `--spring.profiles.active=dev`

## Testing

```bash
# Run all tests
./gradlew test

# Run with detailed output
./gradlew test --info

# Run tests in continuous mode (re-run on file changes)
./gradlew test --continuous

# Run a specific test class
./gradlew test --tests "org.jcr.usuariosenmemoria.UsuariosEnMemoriaApplicationTests"
```

Current test coverage: Basic application context load test in `UsuariosEnMemoriaApplicationTests.java`

**Note:** This is an educational project with minimal test coverage. For production projects, implement comprehensive unit tests (Service layer), integration tests (Controller + Service + Repository), and consider contract testing for API endpoints.

## CORS Configuration

CORS is configured in `config/WebConfig.java` to allow cross-origin requests from frontend applications:

**Allowed Origins:**
- `http://localhost:3000` - React/Vue/Next.js development server
- `http://localhost:4200` - Angular development server
- `https://vercel-livid-nine-35.vercel.app` - Production domain (Vercel deployment)
- `https://vercel2-alpha-eosin.vercel.app` - Second production domain (Vercel deployment)

**Configuration:**
- **Paths:** `/api/**` (all API endpoints)
- **Methods:** GET, POST, PUT, DELETE, OPTIONS
- **Headers:** All allowed (*)
- **Credentials:** Enabled (allows cookies and auth headers)
- **Max Age:** 3600 seconds (preflight cache)

⚠️ **IMPORTANT:** Before production deployment, update allowed origins in `WebConfig.java` with actual production domains.

## Security Considerations (Educational Context)

⚠️ **This is a demonstration project for educational purposes.**

Current limitations (NOT suitable for production):
- Passwords stored in **plain text** (no encryption)
- **No authentication/authorization** required for any endpoint
- No HTTPS enforcement
- CORS allows all headers and credentials (too permissive for production)
- No rate limiting
- No audit logging

For production, implement:
- Password encryption (BCrypt/Argon2) via Spring Security
- Authentication/authorization (JWT, OAuth2)
- HTTPS only
- Restrict CORS to specific production domains only
- Rate limiting
- Audit trails
- Persistent database (PostgreSQL, MySQL)

## Package Structure

```
org.jcr.usuariosenmemoria/
├── UsuariosEnMemoriaApplication.java    # Main Spring Boot application class
├── config/                               # Configuration classes
│   ├── OpenApiConfig.java               # Swagger/OpenAPI 3.0 configuration
│   └── WebConfig.java                   # CORS and MVC configuration
├── controller/                           # REST Controllers (Presentation layer)
│   └── UsuarioController.java           # User CRUD endpoints at /api/usuarios
├── service/                              # Business Logic layer
│   └── UsuarioService.java              # User business logic with @Transactional
├── repository/                           # Data Access layer
│   └── UsuarioRepository.java           # JpaRepository<Usuario, Long>
├── model/                                # JPA Entities (Domain models)
│   └── Usuario.java                     # User entity with @Entity, @Builder
├── dto/                                  # Data Transfer Objects (API contracts)
│   ├── UsuarioRequestDTO.java           # Input validation (record type)
│   ├── UsuarioResponseDTO.java          # Output mapping (excludes password)
│   └── ErrorDTO.java                    # Standardized error responses
└── exception/                            # Exception handling
    └── GlobalExceptionHandler.java      # @ControllerAdvice for centralized error handling
```

**Key Points:**
- **Single Domain:** All code in `org.jcr.usuariosenmemoria` package
- **Layer Separation:** Clear separation between controller/service/repository
- **DTO Isolation:** DTOs in separate package to emphasize API contract separation
- **Configuration Package:** Spring configuration classes grouped together
- **No Subpackages:** Flat structure suitable for single-entity educational project

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cortezalberto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
