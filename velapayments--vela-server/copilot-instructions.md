## core-api

> Core backend conventions for the NestJS Vela server


# Backend Core — Vela server

You are a Senior Backend Engineer working on a **NestJS API** with **Prisma** and **Supabase PostgreSQL**.

## Architecture

- **Clean Architecture**: Controllers → Services → PrismaService → Database
- All business logic lives in **Services**, never in Controllers
- Controllers only handle HTTP concerns (params, auth, delegation to service, response)
- Database access is exclusively through **PrismaService** (global module)

## Project Structure

```
src/modules/{feature}/
├── {feature}.module.ts
├── {feature}.controller.ts
├── {feature}.service.ts
├── {feature}.controller.spec.ts
├── {feature}.service.spec.ts
└── dto/
    ├── create-{feature}.dto.ts
    └── update-{feature}.dto.ts
```

## TypeScript Standards

- **Strict TypeScript** — never use `any` (use `unknown` + type narrowing instead)
- Explicitly type all function parameters and return types
- No unused variables, imports, or functions
- No unnecessary comments — code should be self-documenting

## NestJS Conventions

- Use `class-validator` + `class-transformer` for DTOs
- Use `@ApiTags`, `@ApiOperation` on all controllers (Swagger)
- Use custom `@Public()` decorator for unauthenticated endpoints
- Use `@CurrentUser()` decorator to get the authenticated user
- Errors must use NestJS exceptions: `NotFoundException`, `ForbiddenException`, `BadRequestException`, etc.
- Use `EventEmitter2` for cross-module communication (notifications, audit logs)

## Error Handling

- Business validation failures: throw specific NestJS HTTP exceptions (e.g. `BadRequestException`, `ConflictException`)
- Database errors: let Prisma exceptions propagate (handled by global filter)

## Documentation

Detailed docs for each subsystem live in `docs/`. Reference them for deep context:

- `vela-overview.md` — overall product flows and UX
- `server-build-plan.md` — server backend build plan
- `server-build-plan-consolidated.md` — consolidated plan
- `unit-tests.md` — testing patterns and conventions
- `DATABASE.md` — Prisma setup and commands

---
> Source: [VelaPayments/vela-server](https://github.com/VelaPayments/vela-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
