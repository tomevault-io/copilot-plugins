## features-wite-tec

> Stack: NestJS (Node) + .NET 8 + React 18 + PostgreSQL 15 + Redis 7

# WIA-272: Billing Links

Stack: NestJS (Node) + .NET 8 + React 18 + PostgreSQL 15 + Redis 7

## Comandos essenciais
- Testes Node: `cd node-api && npm test`
- Testes .NET: `cd dotnet-service-tests && dotnet test`
- Testes Frontend: `cd frontend && npm test`
- Subir infra: `docker compose up -d`

## Regras criticas
- name/cpf NUNCA em logs — PiiSanitizer obrigatorio
- seller_id sempre do JWT
- Idempotency-Key obrigatorio no endpoint publico

<!-- GSD:project-start source:PROJECT.md -->
## Project

**WIA-272: Billing Links — Production Refactor**

Refatorar o PoC de Billing Links para production-ready. O PoC atual demonstra o fluxo completo (seller cria link, cliente paga via endpoint publico, transacao processada pelo servico .NET), mas usa stack desatualizada (Nest 10 + TypeORM), estrutura fragmentada (3 repositorios separados), e tem gaps criticos de funcionalidade. O objetivo e migrar para a stack real da WiteTec (Nest 11 + Prisma), reorganizar em monorepo src/modules, e completar todos os fluxos para producao.

**Core Value:** Seller cria um billing link e compartilha; cliente acessa `/pay/:slug`, paga via PIX ou cartao, e o seller ve o resultado no dashboard em tempo real.

### Constraints

- **Stack:** Nest 11 + Prisma (alinhamento com WiteTec real) — migrar de Nest 10 + TypeORM
- **Seguranca:** name/cpf NUNCA em logs — PiiSanitizer obrigatorio em todos os paths de erro
- **Autenticacao:** seller_id SEMPRE do JWT token (claim `sub`), nunca do body/query
- **Idempotencia:** Idempotency-Key obrigatorio no endpoint publico, implementacao atomica com SET NX
- **Infraestrutura:** PostgreSQL 15 + Redis 7 (ja existem via Docker Compose)
- **Integracao:** .NET service mantido como PSP gateway — comunicacao via HTTP interno
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- TypeScript 5.3.3 - Node.js API (`node-api/`) and Frontend
- C# 8.0 - .NET service (`dotnet-service/`)
- JavaScript/JSX - React components
- SQL - PostgreSQL schema and migrations
## Runtime
- Node.js (inferred 18.x or 20.x from package.json constraints)
- .NET 8.0 runtime
- npm (Node.js projects)
- NuGet (dotnet dependencies)
- Lockfile: `node-api/package-lock.json` and `frontend/package-lock.json` (standard npm lockfiles)
## Frameworks
- NestJS 10.0.0 - Node.js backend API framework (`node-api/`)
- React 18.2.0 - Frontend UI library (`frontend/`)
- ASP.NET Core 8.0 - .NET Web API framework (`dotnet-service/`)
- Jest 29.7.0 - Node.js unit and integration tests (config: `node-api/jest.config.js`)
- Vitest 1.1.0 - Frontend unit tests (config: `frontend/vite.config.ts`)
- xUnit 2.6.2 - .NET unit tests (config: `dotnet-service-tests/WitetecBillingService.Tests.csproj`)
- Vite 5.0.8 - Frontend bundler and dev server (`frontend/`)
- TypeScript 5.3.3 - TypeScript compiler for both Node and frontend
- ts-jest 29.1.1 - Jest transformer for TypeScript
- ts-node-dev 2.0.0 - Development server for Node API
## Key Dependencies
- `@nestjs/core` 10.0.0 - NestJS core framework
- `@nestjs/jwt` 10.0.0 - JWT token handling
- `@nestjs/passport` 10.0.0 - Authentication middleware
- `@nestjs/typeorm` 10.0.0 - ORM integration
- `typeorm` 0.3.17 - SQL ORM with TypeScript support
- `pg` 8.11.3 - PostgreSQL driver
- `ioredis` 5.3.2 - Redis client library
- `axios` 1.6.0 - HTTP client for .NET service communication
- `passport-jwt` 4.0.1 - JWT strategy for Passport
- `class-validator` 0.14.0 - DTO validation
- `class-transformer` 0.5.1 - DTO transformation
- `uuid` 9.0.0 - UUID generation
- `rxjs` 7.8.1 - Reactive programming
- `react-router-dom` 6.21.0 - Client-side routing
- `axios` 1.6.0 - HTTP client for API calls
- `uuid` 9.0.0 - UUID generation
- `clsx` 2.0.0 - Conditional className builder
- `tailwindcss` 3.4.0 - Utility-first CSS framework
- `autoprefixer` 10.4.16 - PostCSS plugin for vendor prefixes
- `Microsoft.EntityFrameworkCore.Design` 8.0.0 - EF Core design-time tools
- `Npgsql.EntityFrameworkCore.PostgreSQL` 8.0.0 - PostgreSQL provider for EF Core
- `Microsoft.AspNetCore.Authentication.JwtBearer` 8.0.0 - JWT authentication
- `Serilog.AspNetCore` 8.0.0 - Structured logging
- `Serilog.Sinks.Console` 5.0.0 - Serilog console output
## Configuration
- Environment variables via `.env` file (see `.env.example` for template)
- Key configs:
- Node.js: `node-api/tsconfig.json` - TypeScript compilation settings
- Frontend: `frontend/tsconfig.json`, `frontend/vite.config.ts` - Build and TypeScript config
- .NET: `dotnet-service/WitetecBillingService.csproj` - Project definition
## Platform Requirements
- Node.js 18.x or higher (implied by package.json)
- .NET SDK 8.0
- PostgreSQL 15 (via Docker)
- Redis 7 (via Docker)
- Docker and Docker Compose
- Containerized deployment (Docker Compose available)
- PostgreSQL 15 instance
- Redis 7 instance
- Node.js runtime for API
- .NET 8.0 runtime for service
- PostgreSQL 15-alpine (from `docker-compose.yml`)
- Connection via `pg` driver (Node) and `Npgsql` (.NET)
- Migrations in `db/migrations/` applied on container startup
- Redis 7-alpine (from `docker-compose.yml`)
- Client: `ioredis` 5.3.2 on Node.js
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Naming Patterns
- Service files: `[feature].service.ts` (e.g., `billing-links.service.ts`)
- Controller files: `[feature].controller.ts` (e.g., `billing-links.controller.ts`)
- Test files: `[feature].spec.ts` (e.g., `billing-links.service.spec.ts`)
- Entity files: `[feature].entity.ts` (e.g., `billing-link.entity.ts`)
- DTO files: `[feature].dto.ts` placed in `dto/` subdirectory (e.g., `dto/create-billing-link.dto.ts`)
- Module files: `[feature].module.ts` (e.g., `billing-links.module.ts`)
- Guard files: `[feature].guard.ts` (e.g., `jwt-auth.guard.ts`)
- Middleware files: `[feature].middleware.ts` (e.g., `correlation-id.middleware.ts`)
- camelCase for function and method names (e.g., `findAllBySeller()`, `handleCreate()`, `getMetrics()`)
- Async function names use same camelCase convention as sync functions (e.g., `async create()`, `async charge()`)
- Test suite names use `describe()` with human-readable strings, test cases use `it()` with descriptive text
- camelCase for local variables and constants (e.g., `sellerId`, `linkId`, `mockRepo`)
- snake_case for database column names and API response fields (e.g., `seller_id`, `created_at`, `total_approved`)
- UPPERCASE_SNAKE_CASE for constants (e.g., `CORRELATION_ID_HEADER`, `RATE_LIMIT_PER_MINUTE`)
- PascalCase for type and interface names (e.g., `BillingLink`, `BillingLinkStatus`, `CreateBillingLinkDto`, `ChargeResult`)
- Type unions for status fields: `export type BillingLinkStatus = 'active' | 'inactive'` (in `src/billing-links/billing-link.entity.ts`)
## Code Style
- No explicit formatter configured (ESLint available via `npm run lint`)
- 2-space indentation (observed throughout codebase)
- No semicolons at end of statements (observed as pattern, though not enforced)
- Long lines wrapped for readability (e.g., grid template definitions in React components)
- ESLint configured in `node-api/package.json` with script: `eslint "src/**/*.ts" "test/**/*.ts"`
- Frontend lint script: `eslint src --ext ts,tsx --report-unused-disable-directives --max-warnings 0`
- No `.eslintrc` file found; rules use default ESLint configuration
## Import Organization
- Not detected. Imports use relative paths (e.g., `../billing-links/billing-links.service`)
- Absolute imports from `src/` not used
## Error Handling
- NestJS HttpException for HTTP-level errors: `throw new HttpException({ error: 'payment_processor_unavailable', correlationId }, HttpStatus.SERVICE_UNAVAILABLE)` (in `src/public-charge/public-charge.service.ts`)
- NestJS NotFoundException for 404 errors: `throw new NotFoundException('billing_link_not_found')` (in `src/billing-links/billing-links.service.ts`)
- UnauthorizedException for auth failures: `throw new UnauthorizedException('invalid_token')` (in `src/shared/auth/jwt-auth.guard.ts`)
- Error responses include string error codes (not just messages): `{ error: 'rate_limit_exceeded', retry_after: 60 }`
- HTTP status codes set explicitly: `@HttpCode(200)` for DELETE endpoints that return data (in `src/billing-links/billing-links.controller.ts`)
- Errors involving PII must use PiiSanitizer before logging: `const safeBody = PiiSanitizer.safeBody(payload as any)` (in `src/public-charge/public-charge.service.ts`)
- No explicit error logging middleware detected; errors thrown to NestJS exception filters
## Logging
- Bootstrap startup message: `console.log('Node API running on port ${port}')` (in `src/main.ts`)
- **CRITICAL:** Never log PII fields (name, cpf, payerName, payerCpf). Use PiiSanitizer utilities.
- PiiSanitizer methods: `PiiSanitizer.sanitize(object)` returns sanitized copy, `PiiSanitizer.safeBody(object)` returns safe JSON string
## Comments
- Technical debt markers: `// TODO: TECH_LEAD_REVIEW — [explanation]` (in `src/billing-links/billing-links.service.ts`)
- Complex business logic requiring explanation
- Not used for obvious code
- Not consistently used throughout codebase
- Type information expressed via TypeScript types rather than comments
## Function Design
- Service methods typically 5-20 lines
- Async operations follow try-catch pattern for external service calls
- Methods extract concerns into separate functions (e.g., `findByIdAndSeller()` reused by `update()` and `inactivate()`)
- Functions accept DTOs for request payloads: `async create(sellerId: string, dto: CreateBillingLinkDto): Promise<BillingLink>`
- seller_id always required as first or dedicated parameter, extracted from JWT token in controllers
- Optional/configuration parameters passed as objects: `{ timeout: 10000, headers: { 'x-correlation-id': correlationId } }`
- Async functions return Promises: `Promise<BillingLink>`, `Promise<BillingLink[]>`, `Promise<ChargeResult>`
- Null for "not found" results in service layer (checked by controller): `async findActiveById(id: string): Promise<BillingLink | null>`
- Exceptions thrown instead of error return values
## Module Design
- NestJS modules export one controller and one service: `BillingLinksController`, `BillingLinksService`
- Module registers in `@Module()` decorator: `imports: [TypeOrmModule.forFeature([BillingLink])]`
- Services injected via constructor: `constructor(private readonly service: BillingLinksService)`
- Not used in this codebase
- All imports explicit to target files
## Entity/DTO Patterns
- TypeORM entities use decorators: `@Entity('table_name')`, `@PrimaryGeneratedColumn('uuid')`, `@Column()`
- Column names explicitly mapped: `@Column({ name: 'seller_id', type: 'uuid' })`
- Timestamps: `@CreateDateColumn()`, `@UpdateDateColumn()` with snake_case names (`created_at`, `updated_at`)
- class-validator decorators for validation: `@IsInt()`, `@Min(1)`, `@MaxLength(255)`, `@IsString()`
- Located in `dto/` subdirectories within feature modules
- Separate create and update DTOs: `CreateBillingLinkDto`, `UpdateBillingLinkDto`
## Middleware and Guards
- JwtAuthGuard extends `AuthGuard('jwt')` from passport, validates JWT tokens
- Returns UnauthorizedException on invalid token
- seller_id extracted from JWT payload and accessible via `req.user.sellerId`
- CorrelationIdMiddleware: generates or propagates correlation ID across requests/responses
- RateLimiterMiddleware: enforces rate limits per IP address, returns 429 status code on limit exceeded
- All services use NestJS @Injectable() decorator
- Services injected in controller constructors
- Mock services created in tests using `Test.createTestingModule()` with value providers
## React/Frontend Patterns
- Functional components with hooks (React 18)
- useState for local state management
- useEffect for side effects (data loading)
- Inline status check functions: `function StatusBadge({ status }: { status: 'active' | 'inactive' })`
- Component files use PascalCase: `BillingLinksPage`, `BillingLinksList`
- Event handlers prefixed with `handle`: `handleCreate()`, `handleInactivate()`, `handleCopy()`
- Async operations flag with `creating`, `loading` state for UI feedback
- Centralized axios instance in `src/services/api.ts`
- Token injected via axios interceptor: `config.headers.Authorization = Bearer ${token}`
- API calls wrapped in try-catch with user-facing error messages
- TypeScript strict mode enabled in `tsconfig.json`
- Custom types defined in `src/lib/types.ts`
- Component props typed with interfaces
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## Pattern Overview
- **Service boundary at product/domain divide** — Node-API owns HTTP surface, authentication, rate limiting, idempotency; dotnet-service owns transaction lifecycle and state machine logic
- **Stateless services** — Both backends delegate persistence to PostgreSQL and transient state to Redis
- **Correlation ID tracing** — End-to-end request tracking across two services and multiple stores
- **PII-safe error handling** — Personal data (name, CPF) never persists in logs even on exception
## Layers
- Purpose: Seller dashboard (billing link management) and public payment page (unauthenticated payer interface)
- Location: `frontend/src/`
- Contains: React pages, API service client, UI components, tests
- Depends on: Node-API (`http://localhost:3000`)
- Used by: End users (sellers and payers)
- Purpose: HTTP entry point, authentication enforcement, idempotency guarantee, rate limiting, public surface contract
- Location: `node-api/src/`
- Contains: Controllers, request DTOs, middleware, guard chains, PostgreSQL ORM bindings
- Depends on: PostgreSQL (billing_links table), Redis (idempotency, rate limit counters), dotnet-service (internal transactions)
- Used by: Frontend, external callers via public API
- Purpose: Transaction aggregate enforcement, state machine validation, write-once persistence
- Location: `dotnet-service/src/Domain/`, `dotnet-service/src/Application/`
- Contains: Entity aggregate roots, use cases, repository interface, validation logic
- Depends on: In-memory or EF Core repository (currently in-memory for dev)
- Used by: Node-API via HTTP `/internal/transactions`
- Purpose: Persistence mechanism, external service integration
- Location: `dotnet-service/src/Infrastructure/`
- Contains: Repository implementation (currently in-memory, must replace before production)
- Depends on: Transaction entity contract
- Used by: Application use cases
- `shared/auth/` — JWT verification, seller identity extraction
- `shared/correlation/` — Request correlation ID generation and propagation
- `shared/idempotency/` — Redis-backed idempotency guard (SETNX atomic)
- `shared/rate-limit/` — Per-IP rate limiting with Redis counter
- `shared/pii/` — Automatic scrubbing of personal data from logs
## Data Flow
## Key Abstractions
- Purpose: Represents a shareable payment URL generated by a seller
- Examples: `node-api/src/billing-links/billing-link.entity.ts`
- Pattern: ORM-mapped table entity with lifecycle hooks (createdAt auto-set, updatedAt trigger on DB)
- Properties: id (UUID), sellerId (UUID), amount (cents), description, status (active/inactive), timestamps
- Invariants: seller_id required, status restricted to enum, seller_id indexed for query performance
- Purpose: Core business rule enforcer — only valid state transitions allowed
- Examples: `dotnet-service/src/Domain/Entities/Transaction.cs`
- Pattern: Private constructor, static factory (Create), validated state machine via method guards
- Properties: transactionId, billingLinkId, amount, payerName, payerCpf, payerEmail, payerPhone, status, metadata, createdAt, updatedAt
- Invariants: Only Pending→Approved or Pending→Failed allowed; transition violations throw InvalidTransactionTransitionException
- Purpose: Atomically store and retrieve request results keyed by Idempotency-Key header
- Examples: `node-api/src/shared/idempotency/idempotency.service.ts`
- Pattern: Redis SETNX (set-if-not-exists) for atomic check-or-create; TTL expiration for auto-cleanup
- Key space: `idempotency:charge:<Idempotency-Key>`
- Behavior: checkOrSave() returns null if new, returns cached result if duplicate
- Purpose: Enforce per-minute charge request limit per IP+link pair
- Examples: `node-api/src/shared/rate-limit/rate-limiter.middleware.ts`
- Pattern: Redis INCR counter with TTL; reject if counter > limit
- Key space: `rate:charge:<IP>`
- Behavior: Increments counter, sets 60-second expiration on first use; rejects at 31st request (limit=30)
- Purpose: Generate or propagate x-correlation-id across request/response and downstream calls
- Examples: `node-api/src/shared/correlation/correlation-id.middleware.ts`
- Pattern: Middleware extracts from request headers or generates UUID; attaches to req object; injects into outbound HTTP headers
- Behavior: Ensures single ID tracks request through all services and log entries
- Purpose: Strip personal data fields (name, cpf, payerName, payerCpf, pan, cvv) before logging
- Examples: `node-api/src/shared/pii/pii-sanitizer.ts`
- Pattern: Static utility with field redaction list; used in all catch blocks
- Behavior: Replaces PII field values with '[REDACTED]' before serializing to log/error response
## Entry Points
- Location: `node-api/src/main.ts`
- Triggers: `npm run start:dev` or production start script
- Responsibilities: NestJS bootstrap, ValidationPipe registration (whitelist=true, forbidNonWhitelisted=true), listen on PORT (default 3000)
- Location: `frontend/src/main.tsx`
- Triggers: `npm run dev` or build
- Responsibilities: React root mount, router setup with two routes (/ → BillingLinksPage, /pay/:linkId → PublicChargePage)
- Location: `dotnet-service/` (Program.cs, implicit in .NET project)
- Triggers: `dotnet run` or published binary execution
- Responsibilities: ASP.NET Core startup, middleware registration, controller discovery, listen on port 5001
- Location: `node-api/src/billing-links/billing-links.controller.ts`
- Routes:
- Responsibilities: Extract sellerId from JWT, delegate to service, validate access control
- Location: `node-api/src/public-charge/public-charge.controller.ts`
- Routes:
- Responsibilities: Enforce Idempotency-Key header, validate input DTO, coordinate with IdempotencyService and PublicChargeService
- Location: `dotnet-service/src/API/Controllers/InternalTransactionController.cs`
- Routes: POST `/internal/transactions` (internal only)
- Responsibilities: Extract correlation ID from header, invoke CreateTransactionUseCase, return 201
## Error Handling
- **Authentication failures** — JwtAuthGuard.handleRequest() throws UnauthorizedException if token invalid
- **Seller isolation** — Service layer findByIdAndSeller() throws NotFoundException if link not found or seller_id mismatch (guards all PATCH/DELETE)
- **Invalid state transitions** — Transaction.Approve()/Fail() throw InvalidTransactionTransitionException if pre-condition fails
- **Validation errors** — NestJS ValidationPipe rejects DTO whitelist violations with 400
- **Rate limit exceeded** — RateLimiterMiddleware returns 429 before reaching controller
- **Idempotency** — PublicChargeController returns 409 with idempotent=true if key exists
- **External service failure** — PublicChargeService catches axios errors, PiiSanitizer.safeBody() redacts payload, returns 503 with correlation ID
- **PII protection** — Every catch block in service layer calls PiiSanitizer before returning error response
## Cross-Cutting Concerns
- Frontend: Client-side form validation (CPF format, required fields)
- Node-API: ValidationPipe (DTO class-validator decorators) rejects non-whitelisted fields at middleware layer
- dotnet-service: Implicit via domain aggregate — invalid state throws exception
- JWT verification on all authenticated routes via Passport strategy (`JwtStrategy`)
- Seller ID extracted from token `sub` claim, injected into request context
- No seller_id read from request body or query params
- Token secret: `JWT_SECRET` env var (default dev-secret-local)
- Row-level: All queries filter `WHERE seller_id = :sellerId`
- Link access: findByIdAndSeller() ensures authenticated seller owns the link before PATCH/DELETE
- Public endpoints: Public charge and public-info have no authentication check
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, or `.github/skills/` with a `SKILL.md` index file.
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
> Source: [Cigano-agi/features-wite-tec](https://github.com/Cigano-agi/features-wite-tec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
