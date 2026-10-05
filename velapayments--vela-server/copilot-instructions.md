## prisma

> Prisma schema and database conventions for core-api


# Prisma & Database Conventions

## Schema Conventions

- Table names use `@@map("snake_case")` — Prisma models use PascalCase
- UUIDs for all primary keys: `@id @default(uuid()) @db.Uuid`
- Always add `created_at` and `updated_at` timestamps
- Use enums for finite states: `ProjectStatus`, `ApplicationStatus`, `TierLevel`, etc.
- Use `BigInt` for GitHub IDs (`github_repo_id`, `github_issue_id`, `github_pr_id`)

## Relations & Cascade

- `onDelete: Cascade` for child records that shouldn't exist without parent (IssueApplication → Issue, Issue → Repository)
- `onDelete: SetNull` for optional associations (Issue → Campaign)
- Always add `@@index` on FK columns and frequently queried fields
- Use `@@unique` for composite natural keys (e.g. `[repository_id, github_issue_number]`)

## Prisma Usage in Services

```typescript
// Transactions for multi-step writes
await this.prisma.$transaction(async (tx) => {
  await tx.model.create({ data: { ... } });
  await tx.otherModel.update({ where: { ... }, data: { ... } });
});

// Use updateMany when the record might not exist (avoids throwing)
await this.prisma.issue.updateMany({
  where: { repository_id: repoId, github_issue_number: issueNum },
  data: { state: 'closed' },
});

// Use deleteMany for safe deletes (no throw if not found)
await this.prisma.issue.deleteMany({
  where: { repository_id: repoId, github_issue_number: issueNum },
});
```

## Commands

| Command | Use |
|---------|-----|
| `npm run prisma:generate` | After schema changes |
| `npm run prisma:migrate` | Create migration (production) |
| `npm run prisma:push` | Push schema (dev only) |
| `npm run prisma:studio` | Visual DB browser |
| `npm run prisma:seed` | Seed data (TierDefinitions, categories) |

See `docs/DATABASE.md` for full setup guide.

---
> Source: [VelaPayments/vela-server](https://github.com/VelaPayments/vela-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
