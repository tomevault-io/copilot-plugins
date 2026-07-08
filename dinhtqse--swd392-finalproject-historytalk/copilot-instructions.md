## swd392-finalproject-historytalk

> - **Stack**: Spring Boot 3.2 / Java 21 / PostgreSQL / Hibernate 6.3 / Maven

# History Talk Backend – AI Guide

## Project Overview
- **Stack**: Spring Boot 3.2 / Java 21 / PostgreSQL / Hibernate 6.3 / Maven
- **Root**: `Source-code/SWD392_FinalProject_HistoryTalk/history-talk-backend`
- **Context path**: `/Historical-tell` — all API paths are `/Historical-tell/v1/...`
- **Swagger UI**: `http://localhost:8080/Historical-tell/api/v1/swagger-ui`
- **Layer order**: Controller → Service → Repository, with DTOs between every layer boundary

## Package Structure
```
com.historyTalk
├── config/          # SecurityConfig, SpringSecurityConfig, OpenApiConfig, SwaggerConfig
├── controller/
│   ├── authentication/   # AuthController
│   ├── character/        # CharacterController
│   ├── chat/             # ChatController
│   └── historicalContext/ # HistoricalContextController, HistoricalContextDocumentController
├── dto/
│   ├── authentication/   # LoginRequest/Response, RegisterRequest/Response, RefreshTokenResponse
│   ├── chat/             # CreateChatSessionRequest, ChatSessionResponse,
│   │                     # MessageResponse, GetMessagesResponse, SendMessageRequest,
│   │                     # SendMessageResponse, ChatHistorySessionItem, ChatHistoryGroupResponse
│   ├── character/        # Create/Update/Response DTOs for character
│   ├── historicalContext/ # Create/Update/Response DTOs for context and document
│   ├── exception/        # InvalidArgumentResponse
│   ├── user/             # UserInformationResponse
│   ├── ApiResponse.java
│   ├── PaginatedResponse.java
│   └── ValidationErrorResponse.java
├── entity/
│   ├── ContextStatus.java (enum – unused, kept for future)
│   ├── enums/     # UserRole (enum: CUSTOMER | STAFF | ADMIN)
│   ├── user/      # User (has UserRole role directly)
│   ├── historicalContext/ # HistoricalContext, HistoricalContextDocument
│   ├── character/ # Character, CharacterDocument
│   ├── chat/      # ChatSession, Message
│   └── quiz/      # Quiz, Question, QuizResult, QuizAnswerDetail
├── exception/
│   ├── BaseException.java (abstract, holds errorCode + HttpStatus)
│   ├── ResourceNotFoundException.java  → 404
│   ├── DataConflictException.java      → 409
│   ├── InvalidRequestException.java    → 400
│   ├── UnauthorizedException.java      → 401
│   ├── SystemException.java            → 500
│   ├── BusinessRuleViolationException  → RuntimeException (uncategorized rule breaks)
│   └── GlobalExceptionHandler.java
├── repository/    # HistoricalContextRepository, HistoricalContextDocumentRepository,
│                  # UserRepository, CharacterRepository, ChatSessionRepository,
│                  # MessageRepository
├── security/      # JwtAuthenticationFilter, JwtTokenProvider, UserPrincipal, AuthenticatedPrincipal
├── service/
│   ├── authentication/ # AuthService (interface), AuthServiceImpl, JwtService, JwtServiceImpl,
│   │                   # CustomUserDetailsService
│   ├── character/      # CharacterService
│   ├── chat/           # ChatSessionService, MessageService, ChatHistoryService, AiServiceClient
│   └── historicalContext/ # HistoricalContextService, HistoricalContextDocumentService
├── mapper/        # (character/, historicalContext/, user/ sub-packages)
└── utils/
    ├── authentication/
    └── SecurityUtils.java  # getUserId(), getRoleName() from SecurityContext
```

## Entity & ID Conventions
- **All primary keys are `UUID` type** with `@GeneratedValue + @UuidGenerator` — Hibernate auto-generates, stored as PostgreSQL native `uuid` column (16 bytes).
- **No `@PrePersist` for ID generation** — only use `@PrePersist` for setting non-null default values (e.g. `isFromAi = false`).
- FK relations use `@ManyToOne` / `@OneToMany(mappedBy=...)` with `FetchType.LAZY`.
- When a service receives an ID from a controller `@PathVariable` (String), convert with `UUID.fromString(id)` before calling repository.
- When mapping entity → DTO, call `.toString()` on UUID fields.

## Exception Hierarchy — Use These, Never Create Ad-hoc
| Class | HTTP | When to use |
|---|---|---|
| `ResourceNotFoundException` | 404 | Entity not found by ID / name |
| `DataConflictException` | 409 | Duplicate name, unique constraint violation |
| `InvalidRequestException` | 400 | Business rule validation failures |
| `UnauthorizedException` | 401 | Bad credentials, invalid/expired token |
| `SystemException` | 500 | Unexpected infrastructure errors |
| `BusinessRuleViolationException` | — (RuntimeException) | Misc rule violations not covered above |

All exceptions extend `BaseException(message, errorCode, httpStatus)` except `BusinessRuleViolationException` which extends `RuntimeException` directly.  
`GlobalExceptionHandler` catches `BaseException` subclasses and returns `ErrorResponse` with `errorCode`, `message`, `status`, `timestamp`. Do NOT return `ApiResponse` for exceptions — use `ErrorResponse`.

## Request/Response Patterns
- All **successful** HTTP responses wrap payloads: `ApiResponse.success(data, "message")` or `ApiResponse.error(message, errorCode)`.
- Paginated list endpoints return `PaginatedResponse<T>` (fields: `content`, `totalElements`, `totalPages`, `currentPage`, `pageSize`, `hasNext`, `hasPrevious`).
- Simple list endpoints (no pagination) return `List<T>` wrapped in `ApiResponse.success(...)`.
- `@Valid` on request DTOs — validation errors flow through `GlobalExceptionHandler.handleMethodArgumentNotValid` → returns `ValidationErrorResponse` with list of `{field, message}`.

## Security & Authentication
- Auth module: `POST /v1/auth/register`, `POST /v1/auth/login`, `POST /v1/auth/refresh`, `POST /v1/auth/logout`.
- JWT stored as Bearer token; `JwtTokenProvider` signs with HS512; `JwtAuthenticationFilter` validates on each request.
- `SecurityConfig` — **shared file, coordinate before editing**:
  - `GET /Historical-tell/v1/**` → public
  - Mutating verbs (`POST/PUT/DELETE /v1/**`) → require authenticated JWT
  - `@PreAuthorize("hasRole('STAFF') or hasRole('ADMIN')")` on write endpoints
- `UserPrincipal` wraps `User` entity for Spring Security; `uid` is stored as `String` (`.toString()` from UUID). Authority is a single `ROLE_<role>` derived from `UserRole`.
- **JWT claims**: `sub` (email), `uid`, `role` (e.g. `"CUSTOMER"`, `"STAFF"`, `"ADMIN"`).
- **`AuthenticatedPrincipal`**: lightweight object set as the `Authentication` principal after JWT validation. Fields: `email`, `uid`, `role`. Populated directly from JWT claims — no DB call.
- **`SecurityUtils`** (`utils/SecurityUtils.java`): use `SecurityUtils.getUserId()` and `SecurityUtils.getRoleName()` in controllers to get caller identity. **Never** read `X-Staff-Id` / `X-Staff-Role` headers in business logic — they can be spoofed.
- `X-Staff-Id` / `X-Staff-Role` headers exist only as a fallback in `JwtAuthenticationFilter` for Swagger testing without a token. Do not add them as `@RequestHeader` parameters on any write endpoint.
- CORS is wide-open (`allowedOrigins = "*"`) with stateless sessions.

## Repository Conventions
- `HistoricalContextRepository`: `findAllSimple(search)` (list, ordered), `findAllWithSearch(search, pageable)` (paginated), `findByCreatedByUid(UUID, Pageable)`.
- `HistoricalContextDocumentRepository`: `search(keyword)`, `findByHistoricalContextContextIdOrderByUploadDateDesc(UUID)`, `findByCreatedByUidOrderByUploadDateDesc(UUID)`.
- `ChatSessionRepository`: `findByUserAndCharacterAndContext(userId, characterId, contextId)` (sorted by `lastMessageAt DESC NULLS LAST`); `findBySessionIdAndUserUid(sessionId, userId)` (ownership check); `findAllByUserUid(userId)` (for chat history).
- `MessageRepository`: `findByChatSessionSessionIdOrderByTimestampAsc(UUID sessionId)`.
- Use `ILIKE` (not `LOWER()`) for case-insensitive search in JPQL — Hibernate 6.3 rejects `LOWER()` on entity path expressions.
- Normalize search input to lowercase in Java before passing to `ILIKE` queries.

## Data & Validation Rules
- `HistoricalContext.name` must be unique (case-insensitive). Use `DataConflictException` for duplicates.
- `HistoricalContextDocument.content` is capped at **10 MB** (validated via UTF-8 byte length). Follow this pattern for any large text fields.
- Ownership checks on mutating ops: get `userId` via `SecurityUtils.getUserId()` and `role` via `SecurityUtils.getRoleName()` — compare `entity.getCreatedBy().getUid()` with `UUID.fromString(userId)`; allow bypass if `role.equalsIgnoreCase("ADMIN")`.
- `User.email` and `User.userName` must be unique (`unique = true` on `@Column`).
- All content entities (`HistoricalContext`, `HistoricalContextDocument`, `Character`, `CharacterDocument`, `Quiz`) use `@ManyToOne User createdBy` with `@JoinColumn(name="created_by")` instead of a Staff FK.
- **Chat session ownership**: `ChatSession` belongs to the `User` who created it. Users can only read/delete their own sessions — no admin override. Delete uses `findBySessionIdAndUserUid` (returns 404 if session not found **or** not owned by caller, to avoid leaking existence).
- `ChatSession.title` defaults to `""`. `ChatSession.lastMessageAt` is updated after every saved message.
- `Message.suggestedQuestions` (TEXT) stores a JSON array string (e.g. `["q1","q2","q3"]`) — only populated on `ASSISTANT` messages. Serialize/deserialize with `ObjectMapper`.
- **AI integration**: `AiServiceClient` (Spring `RestClient`) calls BE-Python at `${AI_SERVICE_URL}`. Methods: `chat(...)` — sync; `generateTitleAsync(...)` — `@Async` fire-and-forget. Always pre-fill `characterData` + `contextData` in the request payload so BE-Python skips its own callback to BE-Java.
- `@EnableAsync` on `HistoryTalkApplication`. Async method must be in a separate Spring bean (already satisfied: `AiServiceClient` is `@Service`).
- **`AiServiceClient` constructor must inject `RestClient.Builder` bean** (Spring Boot 3.2 auto-configures it with the full `ObjectMapper`). Never use the static `RestClient.builder()` — it produces a bare-bones builder that fails to serialize Java records properly, causing 422 from BE-Python.
- **Inner records in `AiServiceClient`**: all serialized/deserialized inner records must be **package-private** (no access modifier) or `public`. Using `private record` causes Jackson to silently produce a null body.

## Developer Workflow
- **Build**: `mvn clean install` from `history-talk-backend/`
- **Run**: `mvn spring-boot:run`
- **Config**: env vars in `secretKey.properties` (gitignored) — `DB_URL`, `DB_USER`, `DB_PASSWORD`, `DB_SCHEMA`, `JWT_SECRET`, `JWT_EXPIRATION_MS`, `JWT_REFRESH_EXPIRATION_MS`, `AI_SERVICE_URL` (default `http://localhost:8001`).
- **Schema**: `DB_SCHEMA=historical_schema` (must exist in PostgreSQL before first run); `ddl-auto=update`.
- **Shared files** (coordinate before editing): `SecurityConfig.java`, `SpringSecurityConfig.java`, `JwtAuthenticationFilter.java`, `JwtTokenProvider.java`, `GlobalExceptionHandler.java`, `application.properties`, `pom.xml`.

## Adding a New Module
Follow this order: entity → repository → service → controller → DTOs → register route in `SecurityConfig`.
- Entity: use `UUID` PK with `@GeneratedValue + @UuidGenerator`, `FetchType.LAZY` for all relations.
- Repository: extend `JpaRepository<Entity, UUID>`.
- Service: inject repository, throw typed `BaseException` subclasses, map to DTO with `.toString()` on UUIDs.
- Controller: `@RequestMapping("/v1/...")`, accept IDs as `String @PathVariable`, convert to `UUID` before service call.
- Routes: register in `SecurityConfig.authorizeHttpRequests()` following existing GET-public / mutate-authenticated pattern.

---
> Source: [DinhTQSE/SWD392_FinalProject_HistoryTalk](https://github.com/DinhTQSE/SWD392_FinalProject_HistoryTalk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
