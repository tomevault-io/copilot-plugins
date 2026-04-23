## flashy-cards

> Guidelines for database interactions using Drizzle ORM


# Database Interactions

All database interactions in this project **must** use Drizzle ORM with the defined schema and queries.

## Database Configuration

- Database instance: Import `db` from [index.ts](mdc:src/db/index.ts)
- Schema definitions: All tables are defined in [schema.ts](mdc:src/db/schema.ts)

## Rules

1. **Always import the database instance** from `src/db/index.ts`:

   ```typescript
   import { db } from '@/db';
   ```

2. **Always import table schemas** from `src/db/schema.ts`:

   ```typescript
   import { decksTable, cardsTable } from '@/db/schema';
   ```

3. **Use Drizzle query methods** for all database operations:

   - `db.select()` - for SELECT queries
   - `db.insert()` - for INSERT operations
   - `db.update()` - for UPDATE operations
   - `db.delete()` - for DELETE operations

4. **Never use raw SQL queries** unless absolutely necessary and well-documented

5. **Always use the typed schema** to ensure type safety across the application

## Available Tables

- `decksTable` - Contains flashcard decks with user associations
- `cardsTable` - Contains individual flashcards with deck associations

## Example Usage

```typescript
import { db } from '@/db';
import { decksTable, cardsTable } from '@/db/schema';
import { eq } from 'drizzle-orm';

// Query decks for a user
const userDecks = await db
	.select()
	.from(decksTable)
	.where(eq(decksTable.userId, userId));

// Insert a new card
await db.insert(cardsTable).values({
	deckId: deckId,
	front: 'Question',
	back: 'Answer',
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rdmatiu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
