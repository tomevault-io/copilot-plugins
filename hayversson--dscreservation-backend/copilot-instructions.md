## dscreservation-backend

> - **Classes, interfaces, enums**: PascalCase (e.g. `ReservationService`, `TableController`).


# Spring Boot - Java 25 Standards

## Naming (English only)

- **Classes, interfaces, enums**: PascalCase (e.g. `ReservationService`, `TableController`).
- **Methods and attributes**: camelCase (e.g. `findById`, `customerName`).
- **Constants**: UPPER_SNAKE_CASE (e.g. `MAX_TABLES`).
- **Packages**: lowercase (e.g. `com.devjunior.hayverm.reservation.dto`).

All identifiers must be in **English**.

## Javadoc

- Every **public** method must have Javadoc with:
  - Brief description.
  - `@param` for each parameter (if any).
  - `@return` for non-void methods.
  - `@throws` for declared exceptions.
- Public classes and interfaces must have a short class-level Javadoc.
- Keep Javadoc up to date when you change signatures or behavior.

```java
/**
 * Retrieves a reservation by its unique identifier.
 *
 * @param id the reservation ID (must not be null)
 * @return the reservation if found
 * @throws EntityNotFoundException if no reservation exists for the given id
 */
public ReservationResponse findById(Long id) { ... }
```

## DTOs with Java Records

- Use **Java records** for all DTOs (request/response), not classes with getters/setters.
- Prefer immutable DTOs; use `@JsonProperty` or constructor parameter names for JSON mapping.
- Add Javadoc to the record and to each component when the meaning is not obvious.

```java
/**
 * Response DTO for a single reservation.
 *
 * @param id reservation identifier
 * @param customerName name of the customer
 * @param tableId table identifier
 * @param reservationTime scheduled time
 */
public record ReservationResponse(
    Long id,
    String customerName,
    Long tableId,
    LocalDateTime reservationTime
) {}
```

## Java 25 / Modern Java

- Use **records** for DTOs and value objects.
- Prefer **var** for local variables when the type is obvious from the right-hand side.
- Use **pattern matching** for `instanceof` and `switch` when it improves readability.
- Prefer **sequenced collections** (`List`, `SequencedCollection`) and their new methods when relevant.
- Use **text blocks** (`"""..."""`) for multi-line strings.
- Keep **sealed classes/interfaces** in mind for closed hierarchies (e.g. responses or commands).

## Project Structure

Respect this package layout under `com.devjunior.hayverm.reservation`:

```
src/main/java/com/devjunior/hayverm/reservation/
├── BackendReservasApplication.java    # Main class (root package)
├── config/                             # @Configuration, security, etc.
├── controller/                         # @RestController, HTTP entry points
├── dto/
│   ├── request/                        # Request DTOs (records)
│   └── response/                       # Response DTOs (records)
├── entity/                             # JPA @Entity classes
├── repository/                         # JPA repositories (interface only)
├── service/                            # @Service, business logic
└── exception/                          # Custom exceptions, @ControllerAdvice

src/main/resources/
├── application.properties
└── application-*.properties            # Profiles (e.g. dev, prod)

src/test/java/.../                      # Mirror main package for tests
```

- **Controllers**: thin; delegate to services; use DTOs in and out.
- **Services**: contain business logic; use entities and repositories; return DTOs.
- **Repositories**: only Spring Data JPA interfaces in `repository/`.
- **Entities**: in `entity/`; avoid business logic; use for persistence only.
- **DTOs**: only in `dto/request` and `dto/response`; use records.

## General Conventions

- Prefer constructor injection (`@RequiredArgsConstructor` or explicit constructor) over field injection.
- Validate input with Bean Validation (`@Valid`, `@NotNull`, `@Size`, etc.) on request DTOs.
- Do not expose entities directly in API responses; map to response DTOs.
- Handle exceptions in a central `@ControllerAdvice` and return consistent error payloads.
- Use meaningful HTTP status codes and avoid putting business data in generic 500 messages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hayversson)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Hayversson)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
