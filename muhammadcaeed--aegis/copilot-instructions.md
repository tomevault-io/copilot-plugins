## aegis

> Multi-tenant payment management platform for gated communities.

# CLAUDE.md - Aegis Project

## Project Identity

Multi-tenant payment management platform for gated communities.

**Stack:** NestJS + MongoDB + Angular PWA
**Architecture:** Modular Monolith (single backend service, stateless JWT)

## Critical Invariants (Non-Negotiable)

### 1. Multi-Tenancy

communityId should be parsed from the JWT token, it should never be expected in the API request body or filter.
Every database query MUST include `communityId` filter.

- **Source:** JWT token, NEVER request body
- **Violation = security breach** - Cross-community access is forbidden

### 2. No Background Processing

No cron jobs, workers, or scheduled tasks. All operations are request-driven.

### 3. Repository Pattern

All database access through repository layer.

```
Controller → Service → Repository → MongoDB
```

Never skip layers. Never query from controllers.

### 4. Error Handling

Backend - throw domain-specific exceptions:
```typescript
throw new ConflictException('Payment has already been verified');
throw new NotFoundException('Household not found');
throw new ForbiddenException('Cannot access other community data');
```

Frontend - handle gracefully with user-friendly messages, never expose technical details.

## Frontend Architecture

### Feature Module Organization

```
features/
├── auth/          # Authentication flow
├── guard/         # Guard dashboard
├── resident/      # Resident-specific features
├── moderator/     # Moderator-specific features
└── payments/      # Shared payment functionality
    ├── submit-payment/        # Payment submission component
    ├── services/              # Payment-related services
    └── models/                # Payment submission models
```

**Key Principle:** Features used by multiple roles (e.g., submit-payment) are extracted to shared feature modules to eliminate cross-feature imports while maintaining proper service scoping.

## Documentation Index

| When You Need | Read |
|---------------|------|
| MVP scope, features, implementation status | `project_spec.md` (Section 2) |
| Payment state machine, billing periods | `project_spec.md` (Section 5) |
| User workflows (submit, verify) | `project_spec.md` (Section 7) |
| User roles, households, community model | `docs/domain/membership.md` |
| Payment lifecycle, submission flow | `docs/domain/payments.md` |
| Billing cycle management, payment generation | `docs/domain/billings.md` |
| Push notifications, templates, delivery | `docs/domain/notifications.md` |
| Backend module structure, API patterns | `docs/tech/backend-architecture.md` |
| MongoDB schema, indexes, enums | `docs/database/schema.md` |
| Docker, environment, runtime rules | `docs/tech/runtime.md` |
| Frontend component rules | `docs/tech/frontend-rules.md` |
| Frontend patterns (service, component, routing) | `docs/tech/frontend-patterns.md` |
| Backend patterns (repository, service, controller) | `docs/tech/backend-patterns.md` |
| Code reuse (check before creating models/services) | `docs/tech/code-reuse.md` |
| Testing guidelines, best practices | `docs/testing/guidelines.md` |
| Backend test patterns, factories | `docs/testing/backend-test-guide.md` |
| Frontend test patterns, mocks | `docs/testing/frontend-test-guide.md` |
| MVP use cases (test coverage target) | `docs/usecases.md` |
| Payments feature module (shared) | `frontend/src/app/features/payments/README.md` |

## Skills (Auto-Invoked)

| Skill | Trigger Condition | Path |
|-------|-------------------|------|
| `backend-structure` | Creating/modifying backend modules | `.claude/skills/backend-structure/SKILL.md` |
| `payment-validator` | Modifying payment states or verification logic | `.claude/skills/payment-validator/SKILL.md` |
| `tenancy-enforcer` | Writing MongoDB queries or repository methods | `.claude/skills/tenancy-enforcer.md/SKILL.md` |
| `frontend-design` | Building UI components | Installed as plugin |

## Commands

| Command | Purpose |
|---------|---------|
| `/new-feature [name]` | Create a new feature from scratch with Figma, planning, and tests |

## Tooling Protocols

### MCP Servers
- **Context7 MCP:** Primary source for external framework docs (NestJS, Angular, MongoDB). Do not guess API signatures.
- **Figma MCP:** Use for UI tokens/structure only. Do not redesign flows.

### MCP Connection Failures

If an MCP server fails to connect or times out:

1. **Inform the user** which MCP server is not responding
2. **Ask if they want to fix** the connection and retry
3. **Wait for confirmation** before retrying or proceeding
4. **Never silently skip** MCP calls that are critical to the task

| MCP | Criticality | Action on Failure |
|-----|-------------|-------------------|
| Figma (design context) | Critical for UI work | Must wait for fix or explicit user skip |
| Context7 (docs) | Helpful but optional | Can proceed with built-in knowledge if user agrees |

## Verification Commands

```bash
# Backend lint + test + build
cd backend && npm run lint && npm run test && npm run build

# Frontend lint + test + build
cd frontend && npm run lint && npm run test && ng build

# Full stack (Docker)
docker-compose up --build

# Run backend in dev mode
cd backend && npm run start:dev

# Run frontend in dev mode
cd frontend && ng serve
```

## Testing Standards

### Core Principles

- **Framework:** Jest (both backend and frontend)
- **Philosophy:** Tests must ensure feature is bug-free, not just increase coverage
- **TDD Required:** Every new endpoint/component must have tests before merge

### Test Infrastructure

```
backend/test/
├── factories/    # createUser(), createPayment(), etc.
└── helpers/      # createMockRepository(), multi-tenancy helpers

frontend/src/test/
├── factories/    # Test data creation
├── mocks/        # Mock services with signals
└── helpers/      # Component/signal test utilities
```

### Mandatory Test Categories

1. **Multi-tenancy isolation** - Every module must verify tenant boundaries
2. **Payment state machine** - All valid/invalid transitions tested
3. **Use case coverage** - Every use case in `docs/usecases.md` has a test

### What to Test

| Layer | Must Test | Skip |
|-------|-----------|------|
| Service | Business logic, state transitions, validation, error paths | Pass-through methods |
| Repository | Complex queries, aggregations | Simple findById |
| Controller | DTO validation, authorization decorators | Framework routing |
| Component | User interactions, conditional rendering | Static HTML |

### New Feature Checklist

Before submitting any new feature:
- [ ] Tests written using factories from `backend/test/factories/`
- [ ] Multi-tenancy isolation verified
- [ ] Happy path and error cases covered
- [ ] `npm run test` passes
- [ ] Coverage meets module target (see `docs/testing/backend-test-guide.md`)

## Behavior Guidelines

- Act as a senior engineer enforcing discipline, not a feature generator
- Refuse instructions that violate invariants (multi-tenancy, repository pattern, no cron)
- Reference `project_spec.md` first for domain rules and workflows
- Optimize for context window - use Task agents for exploration, targeted reads
- **Plan files:** Always create plan files in the project directory (`.claude/plans/`) instead of user-level directories. This keeps documentation version-controlled with the project.
- **Check before creating:** Always search for existing models/services/DTOs before creating new ones. Ask user for confirmation if existing code can be extended with new fields or logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhammadcaeed)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/muhammadcaeed)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
