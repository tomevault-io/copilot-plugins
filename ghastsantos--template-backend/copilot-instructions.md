## template-backend

> Convenções Prisma e PostgreSQL


# Prisma e Database

- Ao realizar alterações no banco via prisma, utilizar o agent `prima-specialist.md`
- Schema em `prisma/schema.prisma` na raiz (convenção oficial)
- Models em camelCase (PascalCase para nomes de model)
- Usar singleton em `src/infrastructure/database/prisma.ts` — nunca instanciar PrismaClient diretamente
- Migrações: `npm run db:migrate` (não criar SQL manualmente; usar `prisma migrate dev`)
- Gerar client após alterar schema: `npm run db:generate`
- Referência: `prisma/schema.prisma` e modelo `HealthCheck`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ghastsantos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
