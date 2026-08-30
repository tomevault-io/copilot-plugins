## k7-infrastructure

> K7 Infrastructure layer EF Core and DI conventions


# Infrastructure Layer

- EF config: `IEntityTypeConfiguration<T>` in `Database/Context/Data/Configurations/`
- Never configure entities directly in `OnModelCreating`
- DI: `DependencyInjection.cs` with `Add*Services()`; lambda-based resolution
- **Never** call `BuildServiceProvider()` during registration
- Domain events dispatched by `DispatchDomainEventsInterceptor` - no manual dispatch
- Migrations per provider: Postgres or Sqlite project + `-- --Database:Provider <Provider>`

---
> Source: [kaybi-gh/K7](https://github.com/kaybi-gh/K7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
