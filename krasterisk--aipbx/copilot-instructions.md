## backend-nestjs

> Backend NestJS conventions for aiPBX_backend


# Backend NestJS Rules

Repo: `aiPBX_backend` (NestJS 11, Sequelize, PostgreSQL).

Planning: canonical `.planning/` lives in sibling `aiPBX` repo; see `aiPBX_backend/.planning/README.md`.

## Module conventions

- Feature modules in `src/<feature>/` with `.module.ts`, `.controller.ts`, `.service.ts`
- DTOs with `@ApiProperty` for Swagger
- Guards: `JwtAuthGuard`, `RolesGuard`, `ApiTokenGuard` as appropriate

## High-risk areas (require dedicated phase)

- `src/ari/`, `src/rtp-udp-server/`, `src/open-ai/`, `src/non-realtime/`
- `src/billing/`, `src/payments/`, `src/accounting/`
- Tenant scoping: always filter by `vpbxUserId`; cache keys must include tenant id

## Tests

- Unit tests: `*.spec.ts` next to service
- Run: `npm test` from `aiPBX_backend/`
- Add test for every new service method with business logic

## API

- Global prefix `/api`
- Update `aiPBX/.planning/intel/API-MAP.md` when adding endpoints

---
> Source: [krasterisk/aiPBX](https://github.com/krasterisk/aiPBX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
