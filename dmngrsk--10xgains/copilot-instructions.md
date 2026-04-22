## 10xgains

> - Use Supabase for backend services, including authentication and database interactions.

### Backend and Database

- Use Supabase for backend services, including authentication and database interactions.
- Follow Supabase guidelines for security and performance.
- Use Zod schemas to validate data exchanged with the backend.
- The API is implemented as a unified Supabase Edge Function using Hono web framework for routing.
- All API endpoints are served from `supabase/functions/api/` with a single `index.ts` entry point.
- Use entities from the `supabase/functions/api/models/` directory as baseline REST API models.
- Follow the established routing patterns defined in `supabase/functions/api/middleware/routes.ts`.
- Use Deno runtime with import maps defined in `supabase/functions/api/deno.json`.
- When implementing unit tests for specific services, use `vitest` functions. Each test file should be saved in the `{tested-file}.test.ts` file.
- Maintain endpoint documentation in the `supabase/functions/api/README.md` file after implementing new endpoints.
- Use dedicated `*.service.ts` data services for communication in Angular components (implement them when necessary), rather than directly using `@supabase/supabase-js`.

### Unified API Edge Function Architecture

The API uses a unified Supabase Edge Function with Hono web framework, organized as follows:

**1. Main Function Directory (`supabase/functions/api/`)**
   - **`index.ts` (Main Entry Point):**
     - Initializes the Hono app and applies root-level middleware (CORS, Supabase client, telemetry)
     - Imports and uses the main router from `middleware/routes.ts`
     - Uses `Deno.serve` as the entrypoint
   - **`context.ts`:**
     - Defines TypeScript types for the Hono application context
     - Contains variables passed through middleware to handlers (Supabase client, authenticated user, telemetry data)
   - **`deno.json`:**
     - Deno configuration with import maps for dependency management
     - Uses URL-based imports (e.g., `https://deno.land/x/hono@v4.2.7/mod.ts`)

**2. Middleware Directory (`supabase/functions/api/middleware/`)**
   - **`routes.ts` (Core Routing Logic):**
     - Defines all API endpoints using Hono's routing system
     - Maps endpoints to handler functions from the `handlers/` directory
     - Uses modular routing with separate route functions for each resource
     - Current API routes:
       - `/api/profiles` - User profile management
       - `/api/exercises` - Global exercise management
       - `/api/plans` - Training plan management
       - `/api/sessions` - Training session management
       - `/api/health` - Health check endpoint
   - **`auth.ts`, `supabase.ts`, `telemetry.ts`:**
     - Middleware for authentication (`requiredAuthMiddleware`, `optionalAuthMiddleware`)
     - Supabase client initialization
     - Telemetry and request tracking

**3. Handlers Directory (`supabase/functions/api/handlers/`)**
   - Organized by resource with files named by HTTP method and modifier
   - Structure: `{resource}/{method}-{modifier}.ts`
   - Examples:
     - `profiles/get-id.ts`, `profiles/put-id.ts`
     - `exercises/get.ts`, `exercises/post.ts`, `exercises/get-id.ts`
     - `plans/get.ts`, `plans/post.ts`, `plans/get-id.ts`
     - `sessions/get.ts`, `sessions/post.ts`, `sessions/post-complete.ts`
     - `session-sets/get.ts`, `session-sets/patch-complete.ts`
   - Each handler receives Hono context with request, response, and middleware data
   - Must include user ownership validation in database queries using `.eq('user_id', getUserId())`

**4. Repositories Directory (`supabase/functions/api/repositories/`)**
   - Contains data access logic abstracted from handlers
   - Examples: `plan.repository.ts`, `session.repository.ts`
   - Handles complex database operations and business logic
   - Uses Supabase client for database interactions

**5. Services Directory (`supabase/functions/api/services/`)**
   - Contains reusable business logic shared across handlers
   - Examples: `exercise-progressions/exercise-progressions.ts`
   - Implements complex algorithms like weight progression calculations

**6. Models Directory (`supabase/functions/api/models/`)**
   - **`api.types.ts`:** API DTOs and command models (source of truth)
   - **`database.types.ts`:** Database schema definitions (source of truth)
   - These files are automatically copied to Angular frontend via `copy-api.types.js` script

**7. Utils Directory (`supabase/functions/api/utils/`)**
   - Shared utility functions for API helpers, Supabase operations, collections handling
   - `api-helpers.ts`: Error handling and response formatting
   - `supabase.ts`: Database utility functions

### Implementation Guidelines

**Request Lifecycle:**
1. Request hits `Deno.serve` entrypoint in `index.ts`
2. Root-level middleware applied (CORS, Supabase, telemetry)
3. Request passed to main router in `middleware/routes.ts`
4. Hono matches request path to registered route
5. Authentication middleware executed (`requiredAuthMiddleware` or `optionalAuthMiddleware`)
6. Handler function from `handlers/` directory executed
7. Handler may call services from `services/` directory for complex business logic
8. Handler calls repository functions from `repositories/` directory for data operations
9. Handler formats response and returns to client

**Error Handling:**
- Use consistent error response formatting from `utils/api-helpers.ts`
- Return appropriate HTTP status codes (400, 401, 403, 404, 500)
- Include detailed error messages and error types for debugging

**Authentication and Authorization:**
- Use `requiredAuthMiddleware` for protected endpoints
- Always include user ownership validation in database queries
- Use `.eq('user_id', getUserId())` pattern in repository methods
- Return 404 for resources that don't exist or don't belong to the user

**Type Safety:**
- Use strict TypeScript with Deno runtime
- Import types from `models/api.types.ts` and `models/database.types.ts`
- Validate request/response data with Zod schemas where appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dmngrsk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
