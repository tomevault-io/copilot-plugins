## nextjs-server-patterns

> This project follows strict patterns for data handling in Next.js using Server Components, Server Actions, and Zod validation.

# Next.js Server Patterns & Data Validation

This project follows strict patterns for data handling in Next.js using Server Components, Server Actions, and Zod validation.

## Data Retrieval

**All data retrieval must be done via Server Components.**

### ✅ CORRECT: Server Component with data fetching

```typescript
import { auth } from "@clerk/nextjs/server";
import { db } from "@/db";
import { decksTable } from "@/db/schema";
import { eq } from "drizzle-orm";

export default async function DecksPage() {
  const { userId } = await auth();
  
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  const decks = await db.select()
    .from(decksTable)
    .where(eq(decksTable.userId, userId));
  
  return (
    <div>
      {decks.map(deck => (
        <DeckCard key={deck.id} deck={deck} />
      ))}
    </div>
  );
}
```

### ❌ INCORRECT: Client-side data fetching

```typescript
"use client"; // Don't fetch data in client components

export default function DecksPage() {
  const [decks, setDecks] = useState([]);
  
  useEffect(() => {
    fetch('/api/decks').then(/* ... */); // Wrong approach
  }, []);
}
```

## Data Mutations

**All database updates, deletes, and inserts must be done via Server Actions.**

Server Actions should be defined in separate files and marked with `"use server"`.

### ✅ CORRECT: Server Action for mutations

```typescript
"use server";

import { auth } from "@clerk/nextjs/server";
import { db } from "@/db";
import { decksTable } from "@/db/schema";
import { eq, and } from "drizzle-orm";
import { z } from "zod";
import { revalidatePath } from "next/cache";

// Define Zod schema
const createDeckSchema = z.object({
  name: z.string().min(1, "Name is required").max(100),
  description: z.string().optional(),
});

// Define TypeScript type from schema
type CreateDeckInput = z.infer<typeof createDeckSchema>;

export async function createDeck(input: CreateDeckInput) {
  const { userId } = await auth();
  
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  // Validate input with Zod
  const validated = createDeckSchema.parse(input);
  
  const [deck] = await db.insert(decksTable)
    .values({
      ...validated,
      userId,
    })
    .returning();
  
  revalidatePath("/decks");
  return deck;
}

// Update example
const updateDeckSchema = z.object({
  id: z.number(),
  name: z.string().min(1).max(100).optional(),
  description: z.string().optional(),
});

type UpdateDeckInput = z.infer<typeof updateDeckSchema>;

export async function updateDeck(input: UpdateDeckInput) {
  const { userId } = await auth();
  
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  const validated = updateDeckSchema.parse(input);
  const { id, ...updates } = validated;
  
  await db.update(decksTable)
    .set(updates)
    .where(
      and(
        eq(decksTable.id, id),
        eq(decksTable.userId, userId)
      )
    );
  
  revalidatePath("/decks");
}

// Delete example
const deleteDeckSchema = z.object({
  id: z.number(),
});

type DeleteDeckInput = z.infer<typeof deleteDeckSchema>;

export async function deleteDeck(input: DeleteDeckInput) {
  const { userId } = await auth();
  
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  const validated = deleteDeckSchema.parse(input);
  
  await db.delete(decksTable)
    .where(
      and(
        eq(decksTable.id, validated.id),
        eq(decksTable.userId, userId)
      )
    );
  
  revalidatePath("/decks");
}
```

### ❌ INCORRECT: Using FormData without proper typing

```typescript
"use server";

// DON'T DO THIS - FormData is not type-safe
export async function createDeck(formData: FormData) {
  const name = formData.get("name"); // No type safety
  // ...
}
```

## Data Validation Requirements

### 1. Always Use Zod for Validation

Every Server Action must validate its input using Zod schemas:

```typescript
import { z } from "zod";

const mySchema = z.object({
  field1: z.string(),
  field2: z.number(),
  // ... define all fields with validation rules
});
```

### 2. Always Define TypeScript Types

Use `z.infer` to derive TypeScript types from Zod schemas:

```typescript
type MyInput = z.infer<typeof mySchema>;

export async function myAction(input: MyInput) {
  const validated = mySchema.parse(input);
  // ... use validated data
}
```

### 3. Validation Rules

- **Required strings**: Use `.min(1)` to ensure non-empty
- **Optional fields**: Use `.optional()`
- **Numbers**: Validate ranges with `.min()` and `.max()`
- **Emails**: Use `z.string().email()`
- **Enums**: Use `z.enum([...])` for fixed sets of values
- **Custom validation**: Use `.refine()` for complex rules

## Client Components Calling Server Actions

Client components should call server actions for mutations:

```typescript
"use client";

import { createDeck } from "@/actions/deck-actions";
import { useState } from "react";

export function CreateDeckForm() {
  const [isLoading, setIsLoading] = useState(false);
  
  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setIsLoading(true);
    
    const formData = new FormData(e.currentTarget);
    
    // Create properly typed object
    const input = {
      name: formData.get("name") as string,
      description: formData.get("description") as string,
    };
    
    try {
      await createDeck(input);
      // Handle success
    } catch (error) {
      // Handle error
      console.error(error);
    } finally {
      setIsLoading(false);
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="name" required />
      <textarea name="description" />
      <button type="submit" disabled={isLoading}>
        Create Deck
      </button>
    </form>
  );
}
```

## File Organization

Organize server actions in a dedicated directory:

```
src/
├── actions/
│   ├── deck-actions.ts
│   ├── card-actions.ts
│   └── ...
├── app/
│   └── (routes)/
└── ...
```

## Cache Revalidation

After mutations, revalidate affected paths:

```typescript
import { revalidatePath } from "next/cache";

export async function updateDeck(input: UpdateDeckInput) {
  // ... perform update
  
  // Revalidate specific paths
  revalidatePath("/decks");
  revalidatePath(`/decks/${input.id}`);
}
```

## Error Handling

Server Actions should throw descriptive errors:

```typescript
export async function myAction(input: MyInput) {
  const { userId } = await auth();
  
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  try {
    const validated = mySchema.parse(input);
    // ... perform operation
  } catch (error) {
    if (error instanceof z.ZodError) {
      throw new Error(`Validation failed: ${error.message}`);
    }
    throw error;
  }
}
```

## Summary Checklist

✅ **DO:**
- Fetch data in Server Components
- Use Server Actions for all mutations (insert/update/delete)
- Validate all Server Action inputs with Zod
- Define TypeScript types using `z.infer<typeof schema>`
- Use properly typed objects (not FormData) as Server Action parameters
- Revalidate cache after mutations

❌ **DON'T:**
- Fetch data in Client Components
- Use API routes for database operations
- Accept FormData as Server Action parameter type
- Skip validation
- Use untyped data

---

**Related Guidelines:**
- See [drizzle-database.mdc](mdc:.cursor/rules/drizzle-database.mdc) for database interaction patterns
- See [clerk-auth.mdc](mdc:.cursor/rules/clerk-auth.mdc) for authentication and authorization requirements

---
> Source: [Rivexo/FlashyCardyCourse](https://github.com/Rivexo/FlashyCardyCourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
