## shareshelf

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**ShareShelf**

ShareShelf is a community-powered tool library web application that lets neighbors borrow and lend rarely used tools and equipment. Instead of buying items used only a few times a year, members share resources within their community — saving money and reducing waste. The app handles user registration, item listings, borrowing requests, reviews/ratings, and trust scoring.

**Core Value:** Users can discover and borrow tools from neighbors in their community, with a trusted borrowing workflow that protects both lenders and borrowers.

### Constraints

- **Tech Stack**: Spring Boot 3.4.x / Kotlin / Next.js 15 / PostgreSQL — locked
- **Deployment**: Railway (backend) + Vercel (frontend) — infrastructure decided
- **Auth Model**: JWT-based stateless auth — not changing
- **Language**: Backend code in Kotlin, frontend in TypeScript
- **Testing**: JUnit 5 + MockK (backend), Vitest + RTL (frontend), Playwright (E2E)
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- **Kotlin 2.1.0** — Backend application logic, JPA entities, controllers, services. Located in `backend/src/main/kotlin/com/shareshelf/`.
- **TypeScript ~5.7** — Frontend application logic, component props, API type definitions. Located in `frontend/src/`.
- **Java 21** — Underlying JVM runtime. No hand-written Java source files found; all backend code is Kotlin compiled to JVM bytecode via `jvmToolchain(21)`.
## Runtime
- **JVM** (eclipse-temurin:21-jre-jammy in production) — Backend runs as a Spring Boot fat JAR (`shareshelf-0.0.1-SNAPSHOT.jar`).
- **Node.js** (via Vercel) — Frontend is built statically or server-side rendered by Next.js.
- **Gradle 8.12** — Kotlin DSL (`settings.gradle.kts`, `build.gradle.kts`). Wrapper configured in `backend/gradle/wrapper/gradle-wrapper.properties`.
- **npm** — `frontend/package.json` present, no lockfile committed (neither `package-lock.json` nor `yarn.lock` or `pnpm-lock.yaml` detected). Dependency versions are semver-ranged.
## Frameworks
- **Spring Boot 3.4.3** — Java/Kotlin web framework. Application entry point at `backend/src/main/kotlin/com/shareshelf/ShareShelfApplication.kt`.
- **Next.js 15 (App Router)** — React meta-framework. Configuration at `frontend/next.config.ts`. All pages are client components under `frontend/src/app/`.
- **React 19** — UI component library. Client components only (no Server Components used in pages found).
- **JUnit 5** (via `spring-boot-starter-test`) — Test runner.
- **MockK 1.13.14** — Kotlin-native mocking library.
- **SpringMockK 4.0.2** — Spring Boot integration for MockK.
- **Spring Security Test** — `@WithMockUser` / `SecurityMockServerRequest` support.
- **Spring Boot DevTools** — Hot reload during development.
- **Kotlin JPA plugin** — `kotlin("plugin.jpa")` for no-arg constructors on entities.
- **Kotlin AllOpen plugin** — `kotlin("plugin.allopen")` to open JPA-annotated classes.
- **Kotlin Spring plugin** — `kotlin("plugin.spring")` to open `@Configuration`/`@Bean` classes.
## Key Dependencies
- **jjwt 0.12.6** (`io.jsonwebtoken:jjwt-api`) — JWT token generation and validation at `backend/src/main/kotlin/com/shareshelf/auth/JwtTokenProvider.kt`. Well-integrated, uses `Keys.hmacShaKeyFor()` with HMAC-SHA.
- **Springdoc OpenAPI 2.8.5** (`springdoc-openapi-starter-webmvc-ui:2.8.5`) — Swagger UI at `/swagger-ui.html` and OpenAPI spec at `/api-docs`. Enabled in dev, disabled in railway profile (`application-railway.yml`).
- **Axios 1.7.9** — HTTP client for frontend API calls. Singleton instance at `frontend/src/lib/api.ts` with request/response interceptors for JWT injection and 401 handling.
- **Flyway** (`flyway-core`, `flyway-database-postgresql`) — Database migrations. Migrations at `backend/src/main/resources/db/migration/V1`-`V5`. `ddl-auto: validate` ensures schema matches migrations.
- **PostgreSQL JDBC Driver** (`org.postgresql:postgresql`) — Database connectivity. Runtime scope.
## Configuration
- Backend uses Spring profiles: `default` (dev at `application.yml` + `application-dev.yml` active by default?), and `railway` (`application-railway.yml` activated in Docker by `SPRING_PROFILES_ACTIVE=railway`).
- Frontend uses `NEXT_PUBLIC_API_URL` environment variable (defaults to `/api` proxy in `frontend/src/lib/api.ts`).
| Variable | Purpose | Set in |
|---|---|---|
| `JWT_SECRET` | HMAC-SHA key for signing tokens | `application.yml` (fallback dev value) |
| `DB_PASSWORD` | Local dev database password | `application.yml` (fallback `shareshelf_dev`) |
| `PGHOST`, `PGPORT`, `PGDATABASE`, `PGUSER`, `PGPASSWORD` | Railway Postgres connection | `application-railway.yml` |
| `PORT` | Server port on Railway | `application-railway.yml` (defaults 8080) |
| `NEXT_PUBLIC_API_URL` | Backend URL for frontend API calls | Frontend env at deploy |
- Backend: `backend/build.gradle.kts` (Gradle Kotlin DSL).
- Frontend: `frontend/next.config.ts`, `frontend/postcss.config.mjs`.
- Docker: `Dockerfile` at project root (multi-stage build using `gradle:8.12-jdk21` then `eclipse-temurin:21-jre-jammy`).
## Platform Requirements
- JDK 21 (tested with Eclipse Temurin).
- Kotlin 2.1.0 compiler (via Gradle plugin).
- Node.js 18+ (for frontend dev server).
- PostgreSQL server on localhost:5432 with database `shareshelf` and user `shareshelf`.
- Gradle 8.12 wrapper (auto-downloaded).
- Railway (backend): Docker container with JDK 21 runtime.
- Vercel (frontend): Node.js runtime, Next.js build output.
- PostgreSQL: Managed by Railway add-on.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Backend: Kotlin / Spring Boot
### Language Features
- Entities: `User` (`auth/entity/User.kt`), `Item` (`item/entity/Item.kt`), `BorrowRequest` (`borrow/entity/BorrowRequest.kt`)
- DTOs: `CreateItemRequest`, `UpdateItemRequest`, `ItemResponse` (`item/dto/ItemDtos.kt`)
- API wrapper: `ApiResponse<T>` (`common/ApiResponse.kt`)
- Database nullable columns use `Type?` (e.g., `val community: String? = null` in `User.kt:22`, `val description: String? = null` in `Item.kt:23`)
- DTO fields that are optional in requests use nullable types (`val description: String? = null` in `CreateItemRequest`)
- Entity IDs use `val id: Long? = null` with auto-generation; non-null asserted via `!!` when used (e.g., `item.id!!` at `ItemService.kt:54`)
- Optional API query parameters use `@RequestParam(required = false)` with nullable types
- Entity fields exposed directly (no getter/setter wrappers); `var` for mutable columns, `val` for immutable ones
- JPA lifecycle callbacks are methods on the entity class (`@PreUpdate`, `@PrePersist`)
### Controller Conventions
- Every controller uses `@RestController` with `@RequestMapping` at class level
- Single constructor injection (no `@Autowired`)
- Methods return `ResponseEntity<ApiResponse<T>>` wrapping all responses
- `@AuthenticationPrincipal principal: UserPrincipal` extracts authenticated user from JWT
- `@RequestBody @Valid` for request body validation
- `@GetMapping` for reads, `@PostMapping` for creates, `@PutMapping` for updates/actions, `@DeleteMapping` for deletes
- Created resources return `ResponseEntity.status(HttpStatus.CREATED).body(ApiResponse.created(...))`
- Successful reads return `ResponseEntity.ok(ApiResponse.success(...))`
- AuthController (`auth/AuthController.kt`) checks `response.success` inline to determine HTTP status
### Service Layer Patterns
- Every service uses `@Service` annotation
- Constructor injection via `private val` parameters (Spring Boot idiomatic Kotlin)
- Dependencies are repository interfaces, utility beans, or other services
- No setter injection or field injection observed
- `@Transactional` applied on write operations (`update`, `delete`, `create` in `BorrowService`)
- Applied at method level, not class level
- Read operations (`findAll`, `findById`) are non-transactional
- `EntityNotFoundException` for missing resources
- `IllegalStateException` for invalid state transitions
- `IllegalArgumentException` for invalid inputs
- `AccessDeniedException` (fully qualified: `org.springframework.security.access.AccessDeniedException`) for authorization failures
### Repository Conventions
- All repositories extend `JpaRepository<Entity, IdType>` 
- No `@Repository` annotation (redundant with Spring Data JPA)
- Query methods use Spring Data JPA naming conventions (`findByOwnerId`, `findByEmail`, `existsByEmail`)
- Custom queries use `@Query` with JPQL and `@Param` annotations
- Return types are either `List<T>`, `T?`, or `Boolean`
### DTO Patterns
- Defined as `data class` in dedicated `dto/` subdirectory
- `@field:` validation annotation syntax (required for Kotlin)
- Optional fields have nullable types with `= null` defaults
- All request DTOs have the field being validated
- Also `data class` with all `val` properties (immutable)
- Contain computed/joined fields (e.g., `ownerName` in `ItemResponse`)
- No validation annotations (output only)
### API Response Wrapper
- Standardized wrapper in `common/ApiResponse.kt`
- `@JsonInclude(NON_NULL)` ensures null fields are omitted from JSON
- Factory methods in companion object
### Error Handling
- `@RestControllerAdvice` class
- Separate `@ExceptionHandler` methods for: `EntityNotFoundException` (404), `IllegalArgumentException` (400), `IllegalStateException` (409), `AccessDeniedException` (403), `MethodArgumentNotValidException` (400 with field errors), general `Exception` (500)
- All return `ResponseEntity<ApiResponse<Unit>>`
- AuthService returns `ApiResponse` directly from service layer instead of throwing exceptions and letting the GlobalExceptionHandler handle them. This means success/failure logic is in the controller (`AuthController.kt:20-25`), unlike other controllers that always succeed at the controller level. This is a mixed approach.
### Import Organization
### Naming Conventions
- Packages: `com.shareshelf.{module}` (e.g., `com.shareshelf.auth`, `com.shareshelf.item`)
- Sub-packages: `entity/`, `dto/` for domain-specific grouping
- Classes: PascalCase (`ItemService`, `AuthController`, `GlobalExceptionHandler`)
- Functions/methods: camelCase (`findAll`, `toResponse`, `parseJsonArray`)
- Properties: camelCase (`dailyPrice`, `depositAmount`)
## Frontend: React / Next.js / TypeScript
### Component Patterns
- All components are default-exported function components
- Props typed via `interface` declared above the component
- Arrow functions not used for component declarations
- Interactive/pages that use hooks start with `"use client";` (e.g., all page files in `app/`, UI components like `Modal.tsx`, `Navbar.tsx`)
- Server components (no directive) used for layout: `layout.tsx`, `page.tsx` (homepage is server component)
- Interfaces defined locally per component file (colocated)
- Extend native HTML attribute interfaces via `React.ButtonHTMLAttributes` / `React.InputHTMLAttributes`
- Optional props use `?` with defaults in destructuring
- `Input.tsx` uses `forwardRef<HTMLInputElement, InputProps>` to support ref forwarding
- Sets `Input.displayName = "Input"` for dev tools
- Local `useState` for component-level state
- `useEffect` for data fetching (no React Query / TanStack Query observed)
- No global state management (Redux, Zustand, etc.)
- `localStorage` for auth token persistence (`lib/auth.ts`)
### File Organization
### Naming Conventions
- Files: PascalCase for components (`Button.tsx`, `ItemCard.tsx`), camelCase for utilities (`auth.ts`, `utils.ts`)
- Functions: camelCase (`handleBorrow`, `formatPrice`, `clearAuth`)
- Interfaces: PascalCase (`ButtonProps`, `Item`, `AuthResponse`)
- Types: PascalCase (`BorrowStatus`, `ItemStatus`)
### Import Organization
### Path Aliases
### API Client
- Axios instance in `lib/api.ts` with interceptors
- Request interceptor injects JWT from `localStorage`
- Response interceptor handles 401 by clearing auth and redirecting to `/login`
- Base URL from `NEXT_PUBLIC_API_URL` environment variable, falls back to `/api`
### Styling
- **Tailwind CSS v4** with `@tailwindcss/postcss`
- Inline Tailwind classes (no styled-components, no CSS modules)
- Consistent color palette: emerald-* for primary actions, stone-* for neutrals
- `className` concatenation pattern: template literals with conditional classes via ternary
### Code Style & Formatting
- No `.eslintrc`, `.prettierrc`, `biome.json`, or `.editorconfig`
- No `ktlint` or `detekt` configuration
- No lint scripts in `package.json` beyond `next lint`
## Inconsistencies and Anti-Patterns
### 1. AuthService returns ApiResponse instead of throwing
### 2. Mixed `needs`+service in AuthController
### 3. Dead `ownerId()` method on Item entity
### 4. Entity fields as `var` with default values vs JPA
### 5. No test files exist
### 6. Direct `ResponseEntity` construction in some controllers
### 7. No linting/formatting tooling
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## System Overview
```text
```
## Component Responsibilities
| Component | Responsibility | File |
|-----------|----------------|------|
| AuthController | Register, login, current user endpoints | `backend/.../auth/AuthController.kt` |
| AuthService | User registration with bcrypt, login authentication, JWT generation | `backend/.../auth/AuthService.kt` |
| JwtTokenProvider | JWT creation, parsing, validation with HMAC-SHA | `backend/.../auth/JwtTokenProvider.kt` |
| JwtAuthenticationFilter | Extract Bearer token, validate, set SecurityContext | `backend/.../auth/JwtAuthenticationFilter.kt` |
| CustomUserDetailsService | Load UserDetails by email or ID for Spring Security | `backend/.../auth/CustomUserDetailsService.kt` |
| SecurityConfig | HTTP security rules, CORS, stateless sessions, filter chain | `backend/.../config/SecurityConfig.kt` |
| CorsConfig | Allow localhost:3000 for development | `backend/.../config/CorsConfig.kt` |
| ItemController | CRUD for items (list, get, create, update, delete) | `backend/.../item/ItemController.kt` |
| ItemService | Item business logic, search, ownership validation | `backend/.../item/ItemService.kt` |
| CategoryController | List categories (read-only) | `backend/.../category/CategoryController.kt` |
| BorrowController | Create/approve/reject/return borrow requests | `backend/.../borrow/BorrowController.kt` |
| BorrowService | Borrow lifecycle, item status transitions | `backend/.../borrow/BorrowService.kt` |
| ReviewController | Create review, list reviews by user | `backend/.../review/ReviewController.kt` |
| ReviewService | Review creation with trust score recalculation | `backend/.../review/ReviewService.kt` |
| ApiResponse | Standardized `{success, message, data, errors}` wrapper | `backend/.../common/ApiResponse.kt` |
| GlobalExceptionHandler | Map exceptions to HTTP status codes with ApiResponse | `backend/.../common/GlobalExceptionHandler.kt` |
| HealthController | Liveness check at `/api/health` | `backend/.../common/HealthController.kt` |
| Axios api client | HTTP client with JWT interceptor and 401 auto-logout | `frontend/src/lib/api.ts` |
| auth.ts lib | localStorage helpers for token and user persistence | `frontend/src/lib/auth.ts` |
## Pattern Overview
- Backend: Package-by-feature organization (`auth/`, `item/`, `borrow/`, `review/`, `category/`, `common/`, `config/`)
- Frontend: Next.js 14 App Router with page-per-route under `src/app/`
- API responses always wrapped in `ApiResponse<T>` for consistency
- Authentication via JWT Bearer tokens in HTTP headers, stateless sessions
- Database migrations via Flyway, schema managed in `db/migration/`
- Frontend uses `"use client"` directives for interactive pages (no server components used)
## Layers
### Frontend Layer:
- Purpose: Browser-side UI rendering with React/Next.js
- Location: `frontend/src/`
- Contains: Pages (app router), components (ui, layout, items), lib (api, auth, utils), types
- Depends on: Backend REST API at `NEXT_PUBLIC_API_URL` (defaults to `/api`)
- Used by: End users in browsers
### Controller Layer:
- Purpose: HTTP request/response handling, route mapping, input validation
- Location: `backend/.../auth/AuthController.kt`, `item/ItemController.kt`, `borrow/BorrowController.kt`, `review/ReviewController.kt`, `category/CategoryController.kt`
- Contains: `@RestController` classes with `@RequestMapping`, `@Valid` for DTO validation
- Depends on: Service classes via constructor injection, DTOs, `ApiResponse`
- Used by: HTTP clients (frontend, curl, mobile apps)
### Service Layer:
- Purpose: Business logic, orchestration, validation, transaction management
- Location: `backend/.../auth/AuthService.kt`, `item/ItemService.kt`, `borrow/BorrowService.kt`, `review/ReviewService.kt`
- Contains: `@Service` classes with injected repositories and other services
- Depends on: Repository interfaces, Entity classes, JWT provider (auth only)
- Used by: Controllers
### Repository Layer:
- Purpose: Data access via Spring Data JPA
- Location: `backend/.../auth/entity/UserRepository.kt`, `item/ItemRepository.kt`, `borrow/BorrowRepository.kt`, `review/ReviewRepository.kt`, `category/CategoryRepository.kt`
- Contains: `interface` extending `JpaRepository<Entity, Long>`, custom `@Query` methods
- Depends on: Entity classes
- Used by: Service classes
### Database Layer:
- Purpose: Persistent storage via PostgreSQL
- Location: `backend/src/main/resources/db/migration/`
- Contains: Flyway migration SQL scripts (V1-V5), seed data for categories
- Depends on: PostgreSQL 15+
- Used by: Repository layer
## Data Flow
### User Registration Flow
### User Login Flow
### Authenticated Request Flow (JWT)
### Item Creation and Browsing Flow
### Borrow Request Lifecycle
### Review Submission Flow
- Frontend: No global state library. Each page uses React `useState` with local component state. Auth persisted in localStorage via `lib/auth.ts`. API calls made directly from pages via `api.ts`.
- Backend: Stateless. All state lives in PostgreSQL database. Authentication state held in JWT token (not server-side session).
## Key Abstractions
- Purpose: Standardized JSON wrapper for every API response
- Examples: `backend/.../common/ApiResponse.kt`
- Pattern: Generic data class with `success`, `message`, `data`, `errors` fields. Static factory methods `success()`, `created()`, `error()`.
- Purpose: Adapter between `User` entity and Spring Security's `UserDetails`
- Example: `backend/.../auth/CustomUserDetailsService.kt:28-39`
- Pattern: Wraps `User` entity, exposes `getId()`, `getEmail()`, `getName()` via custom methods. Used in `@AuthenticationPrincipal` controller parameters.
- Purpose: Separate API contract from entity persistence model
- Examples: `AuthDtos.kt`, `ItemDtos.kt`, `BorrowDtos.kt`, `ReviewDtos.kt`
- Pattern: Each feature package contains a `dto/` sub-package with request/response data classes
- Purpose: Automatic timestamp management
- Examples: `User.kt:47-50`, `Item.kt:44-50`, `BorrowRequest.kt:40-47`, `Review.kt:30-31`
- Pattern: `@PrePersist` and `@PreUpdate` methods set `createdAt` and `updatedAt` timestamps
## Entry Points
- Location: `backend/.../ShareShelfApplication.kt`
- Triggers: JVM startup via `gradle bootRun` or `java -jar`
- Responsibilities: Bootstrap Spring Boot context with all auto-configurations, component scanning
- Location: `backend/.../common/HealthController.kt`
- Triggers: `GET /api/health` HTTP request
- Responsibilities: Return `{"status": "UP"}` for liveness checks
## Architectural Constraints
- **Threading:** Single-threaded per-request model (Spring Boot default with Tomcat). No explicit coroutines or reactive streams used.
- **Global state:** No module-level singletons or shared mutable state. All service beans are stateless singletons managed by Spring DI.
- **Circular imports:** No circular dependency chains detected. Dependency direction is strict: Controller -> Service -> Repository -> Entity with no back-edges.
- **Frontend routing:** All pages use `"use client"` directive. No React Server Components, no Server Actions. Data fetching happens in `useEffect` hooks.
## Anti-Patterns
### N+1 Query Pattern in Service Layer
### Direct Service-to-Repository Calls in CategoryController
## Error Handling
- `EntityNotFoundException` -> `404 NOT_FOUND` with `ApiResponse.error()`
- `IllegalArgumentException` -> `400 BAD_REQUEST`
- `IllegalStateException` -> `409 CONFLICT`
- `AccessDeniedException` -> `403 FORBIDDEN`
- `MethodArgumentNotValidException` -> `400 BAD_REQUEST` with field-level error messages
- `Exception` (catch-all) -> `500 INTERNAL_SERVER_ERROR`
## Cross-Cutting Concerns
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Source: [winnaingsoe6666/ShareShelf](https://github.com/winnaingsoe6666/ShareShelf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
