## gcfm

> This repository follows the Onion Architecture in Go. The high-level directory layout is as follows:

# Contribution Guidelines

This repository follows the Onion Architecture in Go. The high-level directory layout is as follows:

```
./cmd/        - Entry points
./internal/   - Application code
    ├── domain        - Business logic and type definitions
    ├── usecase       - Application use cases
    ├── infrastructure- Implementations of repositories or external systems
    └── interface     - HTTP handlers or other adapters
```

## Layer Responsibilities

- **Domain**: Define entities, value objects and repository interfaces. This layer contains pure business logic and must not depend on any other layer.
- **Usecase**: Describe application-specific workflows. Usecase packages depend only on domain interfaces.
- **Infrastructure**: Provide concrete implementations for DB access or external API calls. Infrastructure packages implement domain repository interfaces.
- **Interface**: Handle input/output (HTTP, CLI, etc.) and orchestrate use cases. Interfaces depend on usecases.

## Dependency Rules

- Dependencies point only from outer layers to inner layers.
- The domain layer is independent of usecase, infrastructure or interface layers.
- Use the Dependency Inversion Principle. Expose interfaces in inner layers and implement them in outer layers.
- Where applicable, inject dependencies via constructors or a dedicated DI mechanism.

This repository adheres to standard Go directory practices. Place new packages under `internal/` unless they are meant for external consumption.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faciam-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/faciam-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
