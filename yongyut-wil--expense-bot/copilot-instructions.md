## expense-bot

> - Use `upsert` instead of manual create/update checks


# Database & Prisma Guidelines

## Prisma Best Practices

- Use `upsert` instead of manual create/update checks
- Always include relevant relations in queries
- Use transactions for multi-step operations
- Add database indexes for frequently queried fields

## Schema Design

- Use `cuid()` for primary keys (not uuid)
- Add `@@unique` constraints where appropriate
- Use `@updatedAt` for tracking changes
- Include meaningful `@@index` for performance

## Query Patterns

- Use `findUnique` for single records by unique field
- Use `findMany` with proper `where` clauses
- Include `select` or `include` to optimize queries
- Use `prisma.$transaction` for atomic operations

## Example

```typescript
// Good - with transaction
await prisma.$transaction(async (tx) => {
  const category = await tx.category.upsert({...})
  const expense = await tx.expense.create({...})
})

// Avoid - multiple separate queries
const category = await prisma.category.create({...})
const expense = await prisma.expense.create({...})
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yongyut-wil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
