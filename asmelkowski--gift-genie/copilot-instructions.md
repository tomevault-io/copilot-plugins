## gift-genie

> - Separate layers: entities, use cases, interfaces, frameworks.

### Architecture — Clean Architecture
- Separate layers: entities, use cases, interfaces, frameworks.
- Dependencies point inward; inner layers have no knowledge of outer layers.
- Domain entities encapsulate business rules without framework dependencies.
- Use interfaces (ports) with adapter implementations for external deps.
- Use cases orchestrate entity interactions for specific operations.
- Use mappers to transform data between layers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/asmelkowski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
