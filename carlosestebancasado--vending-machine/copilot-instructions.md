## vending-machine

> Architecture Principles


# Architecture Principles
- Keep a strict separation by bounded context: do not reference classes from another context unless it is through an explicit contract.
- Follow SOLID, DDD, Hexagonal Architecture, and CQRS: encapsulate domain logic, depend on abstractions, and split commands from queries.
- Place artifacts (entities, documents, repositories, controllers) inside the matching module; do not declare AdminPanel infrastructure under VendingMachine or the other way around.
- When you need new services or adapters, define their interfaces in the right layer and register them through dependency injection.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CarlosEstebanCasado)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CarlosEstebanCasado)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
