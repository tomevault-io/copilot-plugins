## template-nextjs-full-stack

> Rules for Prisma repositories + soft delete

## Prisma Repositories & Soft Delete

- Implement interfaces defined in `src/ports`.
- Use Persistence ↔ Domain mappers from `src/infrastructure/mappers`.
- No business logic; persistence only.
- Must filter `deletedAt: null` by default for reads.
- `archive(id)` = `update({ data: { deletedAt: new Date() } })`; `restore(id)` is the inverse.
- `delete(id)` is reserved for permanent purges.
- Always return Domain entities via the dedicated mapper.

Checklist:
- [ ] `findMany/findUnique` → `where: { deletedAt: null }` where relevant
- [ ] `toDomain(record)` and `toPersistence(entity)` via dedicated mapper
- [ ] Errors thrown in infra must not expose sensitive DB details

Example: [src/infrastructure/prisma/client.repository.prisma.ts](mdc:src/infrastructure/prisma/client.repository.prisma.ts)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SixtySecondsApp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
