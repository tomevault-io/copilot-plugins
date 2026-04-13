## shelfs

> This document provides coding agents with essential context, commands, and conventions for the Shelfs library management system.

# Shelfs Project - Agent Guidelines

This document provides coding agents with essential context, commands, and conventions for the Shelfs library management system.

## Project Overview
- **Tech Stack:** Java 17, Spring Boot 3.2.0, Maven
- **Databases:** PostgreSQL (Production), H2 (Development - in-memory)
- **Base Package:** `com.devaldrete`
- **Main Application:** `src/main/java/com/devaldrete/App.java`

## Build, Test & Run Commands
Use Maven or the provided `Makefile` for standard operations.

### Running & Building
- **Dev (H2 DB):** `mvn spring-boot:run` or `make dev`
- **Prod (PostgreSQL):** `make dev-db` then `mvn spring-boot:run -Dspring-boot.run.profiles=prod`
- **Build (Skip Tests):** `mvn clean package -DskipTests`
- **Clean Build Artifacts:** `mvn clean`
- **Format Code:** `make format`

### Testing
- **Run all tests:** `mvn test` or `make test`
- **Run a single test class:** `mvn test -Dtest=UserServiceTest`
- **Run a single test method:** `mvn test -Dtest=UserServiceTest#testGetUserById`
- **Run tests with coverage:** `mvn clean test jacoco:report`

### Docker Utilities
- **Quick setup:** `make docker-setup`
- **Build & Run:** `make docker-build && make docker-up`

## Code Style & Architecture Guidelines

### Package Structure
- `controller/`: REST endpoints (`@RestController`).
- `service/`: Business logic (`@Service`, `@Transactional`).
- `repository/`: Data access (`@Repository` extending `JpaRepository`).
- `domain/`: JPA entities (`@Entity`).
- `dto/`: Data transfer objects.
- `exception/`: Custom exceptions and handlers.
- `config/`: Configuration classes (Security, Web, etc.).

### Import Order
Group imports in the following order, with a blank line between major groups:
1. `java.*`
2. `jakarta.*` (Note: Spring Boot 3.x uses Jakarta, NOT `javax.*`)
3. Third-party libraries (e.g., `lombok.*`)
4. `org.springframework.*`
5. `com.devaldrete.*`

### Naming & Types
- **Classes/Interfaces:** PascalCase (`UserService`, `BookController`)
- **Methods/Vars:** camelCase (`getUserById`, `userId`)
- **Constants:** UPPER_SNAKE_CASE (`MAX_LOGIN_ATTEMPTS`)
- Return `Optional<T>` from repository queries when a result might be absent.
- Always use `ResponseEntity<T>` as the return type for Controller endpoints.

### Entities (`domain/`)
- Use Lombok annotations: `@Getter`, `@Setter`, `@NoArgsConstructor`.
- Place JPA annotations (`@Id`, `@Column`, etc.) on fields, not getters.
- Use `@Entity` and `@Table(name = "...")`.

```java
@Entity
@Table(name = "users")
@Setter @Getter @NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    @Column(nullable = false, unique = true)
    private String username;
}
```

### Data Transfer Objects (`dto/`)
- **Do NOT use Lombok** for DTOs. Use explicit getters and setters.
- Use Jakarta Bean Validation (`@NotBlank`, `@Email`, `@Size`) with meaningful `message` attributes.

```java
public class UserDTO {
    @NotBlank(message = "Username is required")
    private String username;

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
}
```

### Services & Dependency Injection
- Always use **constructor injection** with `final` fields. Avoid field injection.
- Mark with `@Service` and `@Transactional`.

```java
@Service
@Transactional
public class UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### Error Handling
- Create custom exceptions extending `RuntimeException`.
- Include meaningful context in the exception message.
- Use `@RestControllerAdvice` for global exception handling to return structured JSON.
- Controllers validate incoming request bodies using `@Valid`.

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String resource, String field, Object value) {
        super(String.format("%s not found with %s: '%s'", resource, field, value));
    }
}
```

## API Endpoints & Environments
- **Prefix:** All endpoints are prefixed with `/api` (e.g., `/api/users`).
- **Health Check:** `GET /api/health`
- **Local URLs:**
  - API Base: `http://localhost:8080/api`
  - Swagger UI: `http://localhost:8080/swagger-ui.html`
  - H2 Console: `http://localhost:8080/h2-console`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DevAldrete) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
