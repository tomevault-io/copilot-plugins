## facades

> When writing facade functions, use this guide for facade functions best practices and guidance


# Facade Functions

FacadeConstraints {
  - Apply only to functions in `*-model.ts` files.
  - Function names must follow `<action><Entity><OptionalWith...><DataSource><OptionalBy...>()` pattern.
  - Allowed actions: save | retrieve | update | delete.
  - Entity names are singular, in PascalCase.
  - Use “With…” to indicate included relations before “From/In/ToDatabase”.
  - Use “By…” to indicate lookup key(s) last; key names must match schema fields exactly.
  - Use “And” to chain multiple included relations or keys.
  - Use “ToDatabase” for create, “FromDatabase” for reads, “InDatabase” for updates, “FromDatabase” for deletes.
  - Facades must perform a single database operation (no business logic).
  - Facades must always return raw Prisma results (no transformations).
  - Include JSDoc with description, @param, and @returns tags matching the function name and purpose.
  - Prefer explicit Prisma includes/selects; avoid `include: { *: true }`.
  - Function bodies must use the `prisma.<entity>.<operation>` pattern directly.
}

---
> Source: [janhesters/react-router-saas-template](https://github.com/janhesters/react-router-saas-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
