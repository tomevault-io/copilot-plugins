## crejipsa-be

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CreZipsa — a Spring Boot 3.5.x REST API for a content creator community platform. Java 17, Gradle, MySQL, JWT auth with Kakao OAuth, AWS S3 for file storage, Google Gemini API for AI chat.

## Commands

```bash
./gradlew clean build                    # Full build with tests
./gradlew test                           # Run all tests
./gradlew test --tests "*SomeTest*"      # Run tests by pattern
./gradlew test --tests "tave.crezipsa.crezipsa.SomeTest.someMethod"  # Single method
./gradlew bootRun                        # Run locally (default profile: local)
SPRING_PROFILES_ACTIVE=local ./gradlew bootRun  # Explicit profile
```

No lint/format tasks configured — follow existing file style (tabs, K&R braces).

Test coverage: JaCoCo reports generated to `build/reports/jacoco/test/` after `./gradlew test`.

## Architecture

**Clean / Hexagonal Architecture** with four layers:

| Layer | Package | Responsibility |
|---|---|---|
| **presentation** | `presentation/**/controller` | Thin controllers, return `GlobalResponseDto<T>`, delegate to usecases |
| **application** | `application/**/usecase`, `dto`, `mapper` | Business orchestration, DTOs, mapping. Interfaces (`*Usecase`/`*UseCase`) + `*Impl` |
| **domain** | `domain/**/domain`, `entity`, `repository`, `service` | Entities, value objects, repository port interfaces |
| **infrastructure** | `infrastructure/**/repository`, `client`, `mapper`, `entity` | JPA repos, OAuth clients, S3, Gemini API, entity↔domain mappers |
| **global** | `global/config`, `exception`, `security`, `common` | Cross-cutting: SecurityConfig, JWT, GlobalExceptionHandler, BaseEntity, GlobalResponseDto |

**Port & Adapter flow:** Domain defines repository interfaces (ports) → Infrastructure implements them (adapters via JPA). Use cases depend on domain ports, never on infrastructure directly.

## Key Patterns

**Response envelope:** All endpoints return `GlobalResponseDto<T>` with `{ success, status, message, errorCode, result }`. Use `GlobalResponseDto.success(data)` or `GlobalResponseDto.fail(errorCode)`.

**Error handling:** Throw `CommonException(ErrorCode.XXXX)` for business errors. `GlobalExceptionHandler` catches and wraps. Error codes defined in `global/exception/code/ErrorCode.java` — prefixed by domain (U=User, A=Auth, I=Interest, S=Storyboard, G=Gemini, C=Community/Comment, CH=Chat, SE=Search).

**Entity auditing:** All JPA entities extend `BaseEntity` which provides `createdAt`/`updatedAt` via `@MappedSuperclass`.

**Dual datasource:** Main MySQL (`MainDataSourceConfig`) + analytics MySQL (`AnalyticsJdbcConfig` via `NamedParameterJdbcTemplate`).

**Auth:** JWT filter chain in `global/security/`. Controllers use `@AuthenticationPrincipal User user`. Public endpoints: `/api/auth/**`, `/api/user/signUp`, `/api/auth/refreshToken`.

**Comments:** Support one level of nesting only (depth 0 or 1). No deeper replies.

**Likes:** Composite key `LikeId(userId, communityId)`. Toggle increments/decrements `Community.likeCount`.

## Conventions

- **UseCase naming is inconsistent** (`CommunityUseCase` vs `CommentUsecase`) — preserve existing names, don't normalize.
- Use `@Transactional` on usecases; `@Transactional(readOnly = true)` for queries.
- Use `Pageable` for list endpoints. Watch for N+1 — use batch/join queries.
- External HTTP calls use `WebClient` in `infrastructure/**/client`.
- **Tests:** JUnit 5 + Mockito (`@ExtendWith(MockitoExtension.class)`), AssertJ assertions, H2 in-memory DB for test runtime. Domain-specific fixture classes in `fixture/` package (e.g., `UserFixture`, `ChatFixture`, `StoryboardFixture`) — use `static import` for factory methods. When adding tests for a new domain, create or reuse existing fixtures.
- **Test structure:** Use GWT (Given-When-Then) comment blocks inside each test method. Group tests with `@Nested` by method name, each test annotated with `@DisplayName`.

## Branch Naming

- `feat/#issue/description`
- `refactor/#issue/description`
- `test/#issue/description`


## Development Rules (Token Saving & Efficiency)

Docker-based via GitHub Actions. Configs in `docker/` with dev/prod compose files and `.env` files. CI builds JAR (`-x test`), pushes Docker image, deploys to EC2.
- **Dependency Flow:** Strictly follow `Presentation -> Application -> Domain <- Infrastructure`. Never let Domain depend on any other layer.
- **Entity Policy:** Always use `Mapper` classes to convert between `Domain Model` and `JPA Entity`. Direct leak of JPA Entities to UseCases is prohibited.
- **Import Policy:** Avoid wildcard imports (`import *`). Explicitly import each class.
- **Validation:** Use `jakarta.validation` annotations in DTOs. Business validation should reside in Domain Services or UseCases.

## Frequent Patterns for Agent

- **Adding a new API:** 1. Define `UseCase` interface and `Impl`.
    2. Create/Update `Domain Model` and `Repository Port`.
    3. Implement `JPA Adapter` and `JPA Entity` in infrastructure.
    4. Register `Controller` and use `GlobalResponseDto`.
- **Finding Errors:** Check `global/exception/code/ErrorCode.java` first before creating new error codes.
- **Adding tests:** 1. Create or reuse `Fixture` class in `fixture/` package.
    2. Use `@Nested` per method, GWT comment blocks, `@DisplayName` for each case.
    3. Test both success paths and exception paths (verify `CommonException` with correct `ErrorCode`).

## Implementation Details

- **Lombok:** Use `@Getter`, `@Builder`, and `@RequiredArgsConstructor`. Avoid `@Data` and `@AllArgsConstructor` on Entities.
- **Database:** Use `SnakeCase` for DB columns/tables and `CamelCase` for Java fields.
- **Async:** Use `@Async` with a custom task executor defined in `global/config/AsyncConfig`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CREJIPSA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
