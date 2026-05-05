## 04-product-backend

> Use for backend development — TypeScript API, endpoints, use cases, repositories, Clean Architecture, Vercel deployment


# Backend Developer

> **Read before acting:** `.claude/docs/product/ARCHITECTURE.md`

Build the TypeScript backend for ventures on ReineiraOS using Clean Architecture.

## Architecture Layers

| Layer          | Path                  | Rule                                                |
| -------------- | --------------------- | --------------------------------------------------- |
| Domain         | `src/domain/`         | Business entities, value objects. No external deps.  |
| Application    | `src/application/`    | Use cases, DTOs, mappers. Orchestrates domain.       |
| Infrastructure | `src/infrastructure/` | Repository implementations, external clients.        |
| Interface      | `src/interface/`      | API handlers. Thin — delegates to application.       |
| Core           | `src/core/`           | Cross-cutting: logging, errors, config.              |

## API Conventions

- RESTful: `POST /api/{resource}`, `GET /api/{resource}/:id`
- Validate all input at the boundary
- Consistent errors: `{ error: string, details?: object }`
- Use proper HTTP status codes
- Structured logger, never console.log

## Database

- **DB-agnostic** — repository pattern in infrastructure layer
- Swap Postgres, DynamoDB, Supabase, Turso, or any other store
- Never import DB-specific code outside `src/infrastructure/`

## Commands

```bash
npm run build          # Build TypeScript
npm run test:unit      # Unit tests
npm run test:coverage  # Coverage (80% threshold)
npm run dev            # Local dev server
```

## Checklist

- [ ] Code compiles (`npm run build`)
- [ ] Tests pass (`npm run test:unit`)
- [ ] Input validation on all endpoints
- [ ] No secrets in code (use env vars)
- [ ] Error handling with proper status codes

---
> Source: [ReineiraOS/reineira-atlas](https://github.com/ReineiraOS/reineira-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
