## book-review

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project overview
- Backend API for book reviews built with Spring Boot 3.2, Java 21, Spring Web, Spring Security (JWT), and Spring Data JPA.
- Persistence is PostgreSQL (configured via environment variables in `src/main/resources/application.yaml`).
- Main package: `com.bookapp.book_review`.

## Core commands
Run from repository root.

### Start dependencies
- Start local PostgreSQL:
  - `docker compose up -d`
- Stop local PostgreSQL:
  - `docker compose down`

### Run the app
- Run in dev mode:
  - `./mvnw spring-boot:run`
- Run with explicit env file if needed:
  - `set -a; source .env; set +a; ./mvnw spring-boot:run`

### Build
- Full build:
  - `./mvnw clean package`
- Build without tests:
  - `./mvnw clean package -DskipTests`

### Tests
- Run all tests:
  - `./mvnw test`
- Run a single test class:
  - `./mvnw -Dtest=BookReviewApplicationTests test`
- Run a single test method:
  - `./mvnw -Dtest=BookReviewApplicationTests#contextLoads test`

### Lint / static analysis
- No dedicated lint/checkstyle/spotbugs plugin is configured in `pom.xml`.
- Use compile as a fast sanity check:
  - `./mvnw -DskipTests compile`

### Run packaged jar
- `java -jar target/book-review-api-0.0.1-SNAPSHOT.jar`

## Environment and configuration
- Required environment variables (from `application.yaml`):
  - `DATABASE_URL`
  - `DATABASE_USERNAME`
  - `DATABASE_PASSWORD`
  - `JWT_SECRET` (base64-encoded secret for HMAC signing)
- Optional:
  - `JWT_EXPIRATION` (default `86400000`)
  - `DDL_AUTO` (default `update`)
  - `PORT` (default `8080`)

## High-level architecture
### Request flow and layering
- Controllers in `controller/` define REST endpoints under `/api/**`.
- Controllers delegate business logic to `service/`.
- Services handle validation/business rules and call `repository/` interfaces.
- Repositories use Spring Data JPA plus custom JPQL queries for filtered reads and aggregate calculations.
- DTOs are split into `dto/request` and `dto/response` to keep API contracts separate from entities.

### Security model
- Stateless JWT auth:
  - `security/JwtAuthFilter` extracts `Authorization: Bearer <token>` and sets Spring Security context.
  - `security/JwtUtil` signs/validates JWT using `app.jwt.secret`.
- Route-level access:
  - Public: `/api/auth/**`, GET `/api/books/**`, GET `/api/categories/**`, GET `/api/reviews/**`.
  - All other routes require authentication (`security/SecurityConfig`).
- Method-level authorization:
  - Admin-protected write operations use `@PreAuthorize("hasRole('ADMIN')")` (e.g. category/book writes).

### Domain model and key invariants
- Core entities: `User`, `Book`, `Category`, `Review`, `ReviewLike`.
- Important uniqueness constraints:
  - One review per user per book (`Review` unique constraint on `user_id + book_id`).
  - One like per user per review (`ReviewLike` unique constraint on `user_id + review_id`).
  - Unique usernames/emails/categories and book ISBN.

### Review and rating lifecycle
- `ReviewService` is the center of review workflows:
  - Create/update/delete review.
  - Toggle like/unlike with self-like prevention.
  - Compute `likedByCurrentUser` for list responses in batch (collect review IDs, then query liked IDs once).
- Book stats (`averageRating`, `reviewCount`) are recalculated from review aggregates after review mutations.

### Error handling
- Domain exceptions (`BadRequestException`, `ResourceNotFoundException`, `ForbiddenException`) are converted to structured API responses by `exception/GlobalExceptionHandler`.
- Validation failures (`MethodArgumentNotValidException`) are flattened into a single message string.

### Bootstrap behavior
- `config/DataSeeder` seeds:
  - Default admin account (`admin` / `Admin@123456`) if not present.
  - Default categories if missing.
- Be aware this executes at startup via `CommandLineRunner`.

## API exploration assets
- The `http/` directory contains ready-to-run HTTP client scenarios (`auth.http`, `book.http`, `category.http`, `review.http`) for manual endpoint testing and auth flow verification.

---
> Source: [tinhne/book_review](https://github.com/tinhne/book_review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
