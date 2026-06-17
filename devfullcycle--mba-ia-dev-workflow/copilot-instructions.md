## hexagonal-layers

> Use when writing domain entities, use cases, repositories, controllers, or models. Covers hexagonal architecture dependency rules and layer boundaries.


# Hexagonal Architecture Layers

Each module has two worlds separated by ports: the **domain** (inner hexagon) and the **infrastructure** (outer hexagon). The domain never depends on infrastructure.

## Layer map

```
External world (HTTP, DB, etc.)
  │
  ├── controllers/        → driven ports (incoming adapters)
  ├── models/             → TypeORM entities (infrastructure)
  ├── repositories/       → driving ports implementations (outgoing adapters)
  │
  ── ports ──────────────── boundary ──────────────────
  │
  ├── domain/entities/    → pure domain entities (no framework dependencies)
  ├── domain/services/    → domain services (stateless business logic)
  ├── domain/ports/
  │     ├── driven/       → interfaces for incoming interactions
  │     └── driving/      → interfaces for outgoing interactions (repository contracts)
  │
  └── services/           → use cases (application layer, orchestrates domain)
```

## Rules

### Domain entities vs TypeORM models

- **`domain/entities/`** — pure TypeScript classes/interfaces with business logic. No TypeORM decorators, no framework imports. This is the source of truth for business rules
- **`models/`** — TypeORM entities with decorators (`@Entity`, `@Column`, etc.). These are persistence representations only
- Repositories are responsible for mapping between domain entities and TypeORM models — the domain never knows about TypeORM

### Repository layers

- **`domain/ports/driving/`** — repository interfaces (contracts). Use cases depend on these, never on concrete implementations
- **`repositories/`** — implementations of driving ports using TypeORM. These import TypeORM models and map them to/from domain entities

### Dependency direction (strict)

```
controllers → use cases → domain ports (interfaces) ← repositories (implementations)
                              ↑                              ↓
                        domain entities              TypeORM models
```

- **Use cases** import only domain entities and domain port interfaces — never TypeORM models or concrete repositories
- **Controllers** import only use cases (via container) — never domain entities or repositories directly
- **Repositories** import both domain entities (to return) and TypeORM models (to persist) — they are the bridge
- **Domain entities** import nothing from outside the domain

### Example flow: creating a user

1. **Controller** extracts data from `req.body`, calls use case via container
2. **Use case** receives plain data, creates a domain entity, calls repository port interface
3. **Repository** (implementation) receives domain entity, maps to TypeORM model, persists, maps back to domain entity
4. **Response** flows back: repository → use case → controller → HTTP response (using `uuid`, never `id`)

---
> Source: [devfullcycle/mba-ia-dev-workflow](https://github.com/devfullcycle/mba-ia-dev-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
