## drizzle-database

> Database interaction guidelines using Drizzle ORM


# Database Interaction Guidelines

This project uses **Drizzle ORM** for all database interactions. All database operations must follow these guidelines.

## Schema Definition

The database schema is defined in [src/db/schema.ts](mdc:src/db/schema.ts) and includes:

- **decksTable**: Stores flashcard decks with user associations
- **cardsTable**: Stores individual flashcard cards linked to decks

## Required Practices

### 1. Always Import from Schema

```typescript
import { decksTable, cardsTable } from "@/db/schema";
```

### 2. Use Drizzle Query Methods

All database queries must use Drizzle's query builder or ORM methods:

- **SELECT**: Use `db.select()` or `db.query`
- **INSERT**: Use `db.insert(table).values(...)`
- **UPDATE**: Use `db.update(table).set(...).where(...)`
- **DELETE**: Use `db.delete(table).where(...)`

### 3. Never Use Raw SQL

❌ **DO NOT** write raw SQL queries
✅ **DO** use Drizzle's type-safe query builder

### 4. Type Safety

Leverage Drizzle's TypeScript integration for type-safe database operations. Use `InferSelectModel` and `InferInsertModel` for type definitions:

```typescript
import { type InferSelectModel, type InferInsertModel } from "drizzle-orm";

type Deck = InferSelectModel<typeof decksTable>;
type NewDeck = InferInsertModel<typeof decksTable>;
```

### 5. Relationships

When working with relationships (e.g., cards belonging to decks), use Drizzle's relational queries or proper joins:

```typescript
// Example: Query cards with their deck
db.query.cardsTable.findMany({
  with: {
    deck: true
  }
});
```

## Database Connection

Always use the configured database connection instance (typically imported from a db configuration file) rather than creating new connections.

## Schema Updates

When modifying the database schema:
1. Update [src/db/schema.ts](mdc:src/db/schema.ts)
2. Generate and run migrations using Drizzle Kit
3. Never manually alter the database structure

---

**Remember**: All database interactions must be type-safe, use the Drizzle schema, and leverage Drizzle's query builder methods.

---
> Source: [Rivexo/FlashyCardyCourse](https://github.com/Rivexo/FlashyCardyCourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
