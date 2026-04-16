## hodor

> **Hodor** is a Java Spring Boot authentication service designed to provide robust security and identity management. It leverages modern Java and Spring Boot technologies to deliver a scalable and maintainable authentication solution.

# GEMINI.md - Hodor Project Context

## Project Overview
**Hodor** is a Java Spring Boot authentication service designed to provide robust security and identity management. It leverages modern Java and Spring Boot technologies to deliver a scalable and maintainable authentication solution.

### Core Technologies
- **Java:** 25 (Latest LTS version target)
- **Spring Boot:** 4.0.4 (Spring Boot Starter Parent)
- **Build System:** Maven (using `mvnw` wrapper)
- **Data Persistence:** Spring Data JPA with PostgreSQL
- **Database Migration:** Flyway
- **Security:** Spring Security & Spring Security OAuth2 Authorization Server
- **Monitoring:** Spring Boot Actuator
- **Utilities:** Project Lombok for boilerplate reduction
- **Validation:** Bean Validation (Hibernate Validator)

## Building and Running
The project uses the Maven Wrapper (`mvnw`) for all build and execution tasks.

- **Clean and Build:**
  ```bash
  ./mvnw clean install
  ```
- **Run Application:**
  ```bash
  ./mvnw spring-boot:run
  ```
- **Run with Docker (Required):**
  The project must be capable of running in both local and production environments using Docker.
- **Run Tests:**
  ```bash
  ./mvnw test
  ```
- **Run Specific Test Class:**
  ```bash
  ./mvnw test -Dtest=ClassName
  ```

## Development Conventions

### Package & Code Structure
- **Feature-Based Organization:** Organize code by domain/feature (e.g., `com.calevin.hodor.user`, `com.calevin.hodor.auth`) rather than strictly by technical layer (e.g., `controller`, `service`).
- **Dependency Injection:** Use **Constructor-based Injection** exclusively. Mark dependency fields as `private final` to ensure immutability and testability.
- **DTOs (Data Transfer Objects):** Always use DTOs for API requests and responses. **Never expose JPA entities directly to the API layer.**
- **Lombok Usage:** Prefer `@Data`, `@Value`, `@Builder`, and `@RequiredArgsConstructor` to minimize boilerplate.

### Architecture Layers
- **Controller Layer:** Handle HTTP requests, perform validation using Bean Validation annotations (`@Valid`, `@NotNull`), and delegate to the service layer. Use `@RestController`.
- **Service Layer:** Contain all business logic and transaction boundaries. Use `@Service` and `@Transactional`.
- **Repository Layer:** Use Spring Data JPA interfaces. Extend `JpaRepository` for standard CRUD operations.
- **Domain/Entity Layer:** Standard JPA entities annotated with `@Entity`.

### Testing Strategy
- **Mandatory Requirements:** The project must have tests. Every new functionality added must include unit tests.
- **Unit Testing:** Focus on testing business logic in services using JUnit 5 and Mockito.
- **Slicing Tests:** Use `@WebMvcTest` for controller layer testing and `@DataJpaTest` for repository layer testing.
- **Integration Testing:** Use `@SpringBootTest` for full context integration tests.
- **Testcontainers:** Use Testcontainers for reliable database testing with PostgreSQL.

### Error Handling & Logging
- **Global Exception Handling:** Use `@RestControllerAdvice` and `@ExceptionHandler` to provide consistent error responses across the API.
- **Logging:** Use SLF4J with parameterized messages (e.g., `logger.info("User {} logged in", username);`).

## Key Files
- `pom.xml`: Maven configuration and project dependencies.
- `src/main/java/com/calevin/hodor/HodorApplication.java`: Main application entry point.
- `src/main/resources/application.properties`: Primary application configuration.
- `src/main/resources/db/migration/`: Flyway migration scripts.
- `.agents/skills/java-springboot/SKILL.md`: Detailed Spring Boot development guidelines.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Calevin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
