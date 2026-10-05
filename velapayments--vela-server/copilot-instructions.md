## backend-architecture

> Core API architecture and project structure for Vela


# Core API Architecture — Vela

## Architecture Overview

NestJS + Prisma + Supabase Auth, organized by **feature modules** with global guards.

```
Request → Helmet / CORS → ThrottlerGuard → SupabaseAuthGuard → RolesGuard → Controller → Service → Prisma → PostgreSQL
```

- **Auth is global**: `SupabaseAuthGuard` and `RolesGuard` registered as `APP_GUARD`. No need for `@UseGuards()` on controllers.
- **Public endpoints**: Use `@Public()` decorator.
- **Optional auth**: Use `@OptionalAuth()` to enrich response when token is present.
- **New user flow**: Use `@AllowNewUsers()` for registration endpoints.
- **Role-based access**: Use `@Roles(UserRole.ADMIN)` etc.
- **Events for decoupling**: Use `EventEmitter2` to emit events instead of importing other feature modules directly.

## Project Structure

```
vela-server/
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
│
├── src/
│   ├── main.ts                          # Bootstrap: Helmet, CORS, ValidationPipe, Swagger
│   ├── app.module.ts                    # Root module: ConfigModule, Throttler, EventEmitter, guards
│   │
│   ├── auth/                            # Auth module (self-contained)
│   │   ├── auth.module.ts              # Imports PassportModule, provides SupabaseStrategy
│   │   ├── index.ts                    # Barrel export
│   │   ├── interfaces/
│   │   │   ├── authenticated-user.interface.ts
│   │   │   └── jwt-payload.interface.ts
│   │   ├── strategies/
│   │   │   └── supabase.strategy.ts    # Validates JWT via Supabase, looks up user in DB
│   │   ├── guards/
│   │   │   ├── supabase-auth.guard.ts  # Handles @Public, @OptionalAuth, @AllowNewUsers
│   │   │   └── roles.guard.ts          # Checks @Roles() metadata
│   │   └── decorators/
│   │       ├── public.decorator.ts
│   │       ├── roles.decorator.ts
│   │       ├── current-user.decorator.ts
│   │       ├── allow-new-users.decorator.ts
│   │       └── optional-auth.decorator.ts
│   │
│   ├── database/                        # Global database module
│   │   ├── database.module.ts
│   │   ├── prisma.service.ts           # PrismaClient + pg adapter + lifecycle hooks
│   │   └── index.ts
│   │
│   └── modules/                         # Feature modules
│       ├── users/
│       │   ├── users.module.ts
│       │   ├── users.controller.ts
│       │   ├── users.service.ts
│       │   ├── users.controller.spec.ts
│       │   ├── users.service.spec.ts
│       │   └── dto/
│       │       ├── index.ts
│       │       ├── create-user.dto.ts
│       │       ├── update-user.dto.ts
│       │       └── ...
│       │
│       ├── payment-requests/
│       ├── payments/
│       ├── transactions/
│       └── ...
│
├── test/                                # E2E tests
├── .env.example
├── package.json
├── tsconfig.json
└── prisma.config.ts
```

## Key Conventions

### Feature Module Pattern
Each module is self-contained: `module.ts`, `controller.ts`, `service.ts`, `dto/`, `*.spec.ts`.
Register every new module in `app.module.ts` imports.

### Controller Rules
- Thin controllers: validate access, delegate to service, return result.
- Use `@CurrentUser()` with `AuthenticatedUser` type (never `any`).
- Use `ForbiddenException` for access control, `BadRequestException` for validation.
- All endpoints must have `@ApiOperation`, `@ApiResponse`, `@ApiParam`/`@ApiQuery` for Swagger.

### Service Rules
- All business logic lives in services.
- Throw HTTP exceptions directly (`NotFoundException`, `ConflictException`, etc.).
- Use `EventEmitter2` for side effects (notifications, activity logs, reputation).
- Never import other feature modules — use events for cross-module communication.

### DTO Rules
- Use `class-validator` decorators on every field.
- Use `@ApiProperty` / `@ApiPropertyOptional` for Swagger docs.
- Use `@Transform()` for normalization (trim, lowercase, etc.).

### Imports
```typescript
// Auth decorators/interfaces — always from barrel
import { CurrentUser, Roles, Public, AllowNewUsers, OptionalAuth } from '../../auth';
import { AuthenticatedUser } from '../../auth/interfaces';

// Prisma — from database module
import { PrismaService } from '../../database/prisma.service';

// Prisma enums — from @prisma/client
import { StellarNetwork, PaymentStatus } from '@prisma/client';
```

---
> Source: [VelaPayments/vela-server](https://github.com/VelaPayments/vela-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
