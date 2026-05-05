## clean-architecture

> Enforce Clean Architecture boundaries and dependency direction


# Clean Architecture

- Organize code by layers: `domain`, `application`, `infrastructure`, and `interface`.
- Keep dependencies inward only: outer layers depend on inner layers, never the reverse.
- Domain code contains business rules and must not import frameworks, UI, DB clients, or transport code.
- Use interfaces (ports) in inner layers and provide implementations (adapters) in outer layers.
- Keep use cases in the application layer and make them framework-agnostic.
- Wire dependencies at composition boundaries (startup/DI), not inside domain logic.
- Reject changes that couple business logic to framework-specific concerns.

---
> Source: [fidelrojas-Assure/TaskMaster-Pro](https://github.com/fidelrojas-Assure/TaskMaster-Pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
