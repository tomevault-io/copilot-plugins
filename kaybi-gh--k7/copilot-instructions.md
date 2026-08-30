## k7-application

> K7 Application layer CQRS and validation conventions


# Application Layer

- Request record + handler in **same file**; constructor injection with `private readonly` fields
- Queries: always `AsNoTracking()`
- FluentValidation in `{Name}CommandValidator.cs`; pipeline throws `ValidationException`
- Throw `NotFoundException`, `ForbiddenAccessException` - no error codes or `Result<T>`
- Use `IApplicationDbContext` for DB; `ISender` for cross-feature dispatch
- Domain events: raise in entity, handle in `Features/{Feature}/EventHandlers/`

---
> Source: [kaybi-gh/K7](https://github.com/kaybi-gh/K7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
