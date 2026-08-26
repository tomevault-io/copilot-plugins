## szt-backend

> - This guide is for backend-only work in `backend/` (Spring Boot 3.5, Java 25, Maven).

# AGENTS.md

## Scope
- This guide is for backend-only work in `backend/` (Spring Boot 3.5, Java 25, Maven).
- Code is organized by feature (`recipe`, `comment`, `category`, `user`, `security`, `image`), not by technical layer packages.

## Architecture You Should Preserve
- Default flow is **Controller -> Service -> Repository -> Mapper/DTO** (see `src/main/java/org/glaz/backend/recipe/*`).
- Keep controllers thin and place business rules in services (examples: `RecipeService`, `CommentService`, `AuthService`).
- Paginated endpoints should return `PagedResponse<T>` using `pagination/PagedResponseMapper.java`.
- Keep API error shape consistent through `exception/GlobalExceptionHandler.java` and `exception/GlobalExceptionHandlerFactory.java`.

## Security and Auth Model
- App is stateless JWT: `security/jwt/JwtAuthFilter.java` reads `Authorization: Bearer ...` and sets auth context.
- Endpoint authorization is method-level with `@PreAuthorize` (including SpEL ownership checks like `@recipeService.isOwner(#id)` and `@commentService.isAuthor(#commentId)`).
- Current user lookup goes through `security/auth/AuthContext.java` -> `user/UserService#getCurrentUser()`.

## Domain Rules and Request Contracts
- `User.email` is the primary key (`user/User.java`), and many flows assume email is the stable user ID.
- Recipe-category relation is many-to-many via `recipe_categories`; comments are linked via one-to-many (`recipe/Recipe.java`).
- Recipe filtering combines JPA specifications in `recipe/RecipeSpecs.java` (`containsText`, `hasCategoryNames`, `hasStatus`).
- Recipe create/update endpoints require multipart part names exactly `recipeRq` (JSON) and `image` (`recipe/RecipeController.java`).

## External Integrations
- Images are stored in Google Cloud Storage via `image/ImageStorageService.java` and `gcp.bucket.name`.
- AI recipe generation uses Spring AI + Vertex Gemini in `recipe/AiRecipeService.java` and formats output via `tiptap/TiptapBuilder.java`.
- JWT secret comes from env (`JWT_SECRET`) through `src/main/resources/application.yml`.
- Dev DB uses PostgreSQL + pgvector (`backend/docker-compose.yml`); tests use H2 and disable GCP (`src/test/resources/application.yml`).

## Developer Workflow
- Start local DB:
  - `docker compose -f docker-compose.yml up -d`
- Run API (dev profile):
  - `./mvnw spring-boot:run -Dspring-boot.run.profiles=dev`
- Run tests / package:
  - `./mvnw test`
  - `./mvnw clean package`
- Use JetBrains HTTP smoke tests from `requests/*.http` (`auth.http` populates `accessToken` for later requests).

## Project-Specific Conventions
- Reuse existing exception types from `exception/` instead of custom ad-hoc error responses.
- If editing security/CORS, verify assumptions in `security/SecurityConfig.java` (currently allows `http://localhost:5173`).
- Local startup seeds admin user when missing (`BackendApplication.java`: `admin@test.com` / `test`).

---
> Source: [glazinski/szt-backend](https://github.com/glazinski/szt-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
