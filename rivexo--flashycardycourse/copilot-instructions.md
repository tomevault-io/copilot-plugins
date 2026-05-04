## clerk-auth

> This project uses **Clerk** for authentication and authorization. All data access must be properly scoped to the authenticated user.


# Clerk Authentication & Authorization

This project uses **Clerk** for authentication and authorization. All data access must be properly scoped to the authenticated user.

## Critical Security Requirement

**Users must ONLY be able to access their own data.** Every database query that retrieves user-specific data must be filtered by the authenticated user's ID.

## Getting the Current User

Use Clerk's authentication to get the current user:

```typescript
import { auth } from "@clerk/nextjs/server";

// In Server Components or Server Actions
const { userId } = await auth();

if (!userId) {
  // Handle unauthenticated state
  throw new Error("Unauthorized");
}
```

## Database Query Requirements

### ✅ CORRECT: Always filter by userId

```typescript
// When querying decks
const userDecks = await db.select()
  .from(decksTable)
  .where(eq(decksTable.userId, userId));

// When querying a specific deck
const deck = await db.select()
  .from(decksTable)
  .where(
    and(
      eq(decksTable.id, deckId),
      eq(decksTable.userId, userId)
    )
  );

// When querying cards (through deck ownership)
const cards = await db.select()
  .from(cardsTable)
  .innerJoin(decksTable, eq(cardsTable.deckId, decksTable.id))
  .where(eq(decksTable.userId, userId));
```

### ❌ INCORRECT: Never query without userId filter

```typescript
// SECURITY RISK: Returns all users' data
const allDecks = await db.select().from(decksTable);

// SECURITY RISK: No user verification
const deck = await db.select()
  .from(decksTable)
  .where(eq(decksTable.id, deckId));
```

## Best Practices

1. **Always authenticate first**: Check for `userId` before any data operation
2. **Filter all queries**: Every query for user-specific data must include `userId` filter
3. **Verify ownership**: When accessing related data (like cards), verify the parent resource (deck) belongs to the user
4. **Fail securely**: If no `userId` exists, reject the request immediately
5. **Never trust client input**: Always verify ownership server-side, even if the client claims ownership

## Server Actions & API Routes

All server actions and API routes must follow this pattern:

```typescript
"use server";

import { auth } from "@clerk/nextjs/server";
import { db } from "@/db";
import { decksTable } from "@/db/schema";
import { eq, and } from "drizzle-orm";

export async function getUserDecks() {
  const { userId } = await auth();
  
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  return await db.select()
    .from(decksTable)
    .where(eq(decksTable.userId, userId));
}

export async function updateDeck(deckId: number, data: UpdateDeckData) {
  const { userId } = await auth();
  
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  // Verify ownership before update
  const deck = await db.select()
    .from(decksTable)
    .where(
      and(
        eq(decksTable.id, deckId),
        eq(decksTable.userId, userId)
      )
    );
  
  if (deck.length === 0) {
    throw new Error("Deck not found or unauthorized");
  }
  
  return await db.update(decksTable)
    .set(data)
    .where(
      and(
        eq(decksTable.id, deckId),
        eq(decksTable.userId, userId)
      )
    );
}
```

## Schema Requirements

All tables storing user-specific data must have a `userId` column referencing the Clerk user:

```typescript
export const decksTable = pgTable("decks", {
  id: serial("id").primaryKey(),
  userId: text("user_id").notNull(), // Clerk user ID
  // ... other columns
});
```

## Related Guidelines

- See [drizzle-database.mdc](mdc:.cursor/rules/drizzle-database.mdc) for database interaction patterns
- Combine Clerk authentication with Drizzle's type-safe queries for secure data access

---

**REMEMBER**: Security is not optional. Every data access operation must verify user ownership.

---
> Source: [Rivexo/FlashyCardyCourse](https://github.com/Rivexo/FlashyCardyCourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
