## 92-database

> Rules for database schema and migrations


# 🗃️ Database Rules

> Auto-activated for Prisma, Drizzle, and SQL files.

## Schema Design

### 1. Naming Conventions
```prisma
// ✅ GOOD
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("users")  // snake_case table name
}

// ✅ Relations: explicit names
model Post {
  author   User   @relation("PostAuthor", fields: [authorId], references: [id])
  authorId String @map("author_id")
}
```

### 2. Required Fields (STRICT)
Every table MUST have:
- `id` — Primary key (cuid or uuid preferred)
- `created_at` — Creation timestamp
- `updated_at` — Last update timestamp

For multi-tenant apps:
- `tenant_id` — Tenant isolation (CRITICAL)

### 3. Indexes
```prisma
// ✅ Index frequently queried fields
model Order {
  id        String @id
  userId    String
  status    String
  createdAt DateTime

  @@index([userId])           // FK queries
  @@index([status, createdAt]) // Filtered queries
}
```

## Migration Rules (STRICT)

### 1. Never Edit Existing Migrations
```bash
# ❌ NEVER modify deployed migrations
# ✅ Create new migration for changes
npx prisma migrate dev --name fix_user_email
```

### 2. Reversible Migrations
```sql
-- ✅ Always provide rollback
-- Migration: add_status_column
ALTER TABLE orders ADD COLUMN status VARCHAR(20) DEFAULT 'pending';

-- Rollback:
-- ALTER TABLE orders DROP COLUMN status;
```

### 3. Data Migrations Separate
```
migrations/
├── 001_add_status_column.sql  # Schema only
└── 001_backfill_status.ts     # Data migration (separate)
```

## Query Patterns

### 1. Parameterized Queries (STRICT)
```typescript
// ✅ ALWAYS use parameterized queries
const user = await prisma.user.findUnique({
  where: { email: sanitizedEmail }
});

// ❌ NEVER concatenate SQL
const query = `SELECT * FROM users WHERE email = '${email}'`; // SQL INJECTION!
```

### 2. Select Only Needed Fields
```typescript
// ✅ GOOD: Select specific fields
const users = await prisma.user.findMany({
  select: { id: true, email: true, name: true }
});

// ❌ BAD: Select all (may include sensitive data)
const users = await prisma.user.findMany();
```

### 3. Tenant Isolation (STRICT)
```typescript
// ✅ ALWAYS filter by tenant
const orders = await prisma.order.findMany({
  where: { 
    tenantId: session.tenantId,  // CRITICAL
    status: 'pending'
  }
});
```

## Forbidden Patterns

```typescript
// ❌ NEVER: Raw queries with interpolation
await prisma.$queryRaw`SELECT * FROM users WHERE id = ${userId}`;
// ✅ USE: Prisma.sql for safe interpolation
await prisma.$queryRaw(Prisma.sql`SELECT * FROM users WHERE id = ${userId}`);

// ❌ NEVER: Cascade delete without confirmation
await prisma.user.delete({ where: { id } }); // May delete related data!

// ❌ NEVER: Skip tenant check
await prisma.order.findMany({ where: { status: 'pending' } }); // Cross-tenant leak!
```

## Seeding

```typescript
// seed.ts - for development only
async function seed() {
  // Clear existing data (dev only!)
  if (process.env.NODE_ENV === 'development') {
    await prisma.user.deleteMany();
  }
  
  // Create test data
  await prisma.user.create({
    data: {
      email: 'test@example.com',
      name: 'Test User',
    }
  });
}
```

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
