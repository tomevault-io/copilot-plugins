## flashycardy-ai

> This project uses Clerk for authentication. All user data must be properly secured to ensure users can only access their own data.

# Authentication and Data Access Control with Clerk

This project uses Clerk for authentication. All user data must be properly secured to ensure users can only access their own data.

## Core Principles

1. **Always verify user authentication** before accessing any user-specific data
2. **Use Clerk user ID** as the primary identifier for data ownership
3. **Implement row-level security** in all database queries
4. **Never trust client-side data** - always validate on the server
5. **Use middleware protection** for all protected routes

## Database Schema Requirements

The database schema in [src/db/schema.ts](mdc:src/db/schema.ts) includes `userId` fields that store Clerk user IDs:

- `decksTable.userId` - Owner of the deck
- `userProgressTable.userId` - User's progress data

## Authentication Patterns

### 1. Server-Side Authentication

Always use Clerk's server-side authentication in API routes and server components:

```typescript
import { auth } from "@clerk/nextjs/server";

// In API routes or server components
const { userId } = await auth();
if (!userId) {
  throw new Error("Unauthorized");
}
```

### 2. Client-Side Authentication

Use Clerk's client-side hooks for protected components:

```typescript
import { useAuth } from "@clerk/nextjs";

// In client components
const { userId, isSignedIn } = useAuth();
```

### 3. Route Protection

The middleware in [src/middleware.ts](mdc:src/middleware.ts) protects all routes. For additional protection, use:

```typescript
import { auth } from "@clerk/nextjs/server";
import { redirect } from "next/navigation";

// In server components
const { userId } = await auth();
if (!userId) {
  redirect("/dashboard");
}
```

## Data Access Control Patterns

### 1. Database Queries

Always filter by `userId` in database queries:

```typescript
// ✅ CORRECT - Filter by user ID
const userDecks = await db
  .select()
  .from(decksTable)
  .where(eq(decksTable.userId, userId));

// ❌ WRONG - No user filtering
const allDecks = await db.select().from(decksTable);
```

### 2. API Route Protection

All API routes must verify authentication and ownership:

```typescript
// ✅ CORRECT - Protected API route
export async function GET(request: Request) {
  const { userId } = await auth();
  if (!userId) {
    return new Response("Unauthorized", { status: 401 });
  }
  
  // Always filter by userId
  const userData = await db
    .select()
    .from(decksTable)
    .where(eq(decksTable.userId, userId));
    
  return Response.json(userData);
}
```

### 3. Data Creation

Always associate new data with the authenticated user:

```typescript
// ✅ CORRECT - Associate with user
const newDeck = await db.insert(decksTable).values({
  name: "My Deck",
  userId: userId, // Always include userId
  description: "Description"
});
```

### 4. Data Updates and Deletes

Verify ownership before allowing updates or deletes:

```typescript
// ✅ CORRECT - Verify ownership before update
const deck = await db
  .select()
  .from(decksTable)
  .where(and(
    eq(decksTable.id, deckId),
    eq(decksTable.userId, userId)
  ))
  .limit(1);

if (!deck.length) {
  return new Response("Not found", { status: 404 });
}

// Proceed with update
await db.update(decksTable)
  .set({ name: newName })
  .where(eq(decksTable.id, deckId));
```

## Security Checklist

Before implementing any data access:

- [ ] Verify user is authenticated using `auth()` or `useAuth()`
- [ ] Filter all database queries by `userId`
- [ ] Verify ownership before updates/deletes
- [ ] Use parameterized queries to prevent SQL injection
- [ ] Never expose other users' data in API responses
- [ ] Validate all input data on the server side
- [ ] Use HTTPS in production
- [ ] Implement proper error handling without exposing sensitive information

## Common Anti-Patterns to Avoid

```typescript
// ❌ WRONG - No authentication check
export async function GET() {
  const allData = await db.select().from(decksTable);
  return Response.json(allData);
}

// ❌ WRONG - Trusting client-side data
export async function PUT(request: Request) {
  const { id, userId } = await request.json();
  // Never trust client-provided userId!
  await db.update(decksTable).set({ name: "New Name" }).where(eq(decksTable.id, id));
}

// ❌ WRONG - No ownership verification
export async function DELETE(request: Request) {
  const { id } = await request.json();
  await db.delete(decksTable).where(eq(decksTable.id, id));
}
```

## Testing Authentication

When writing tests or development code:

1. Always mock Clerk authentication
2. Test both authenticated and unauthenticated scenarios
3. Verify that users cannot access other users' data
4. Test edge cases with invalid user IDs

## Error Handling

Implement proper error handling for authentication failures:

```typescript
try {
  const { userId } = await auth();
  if (!userId) {
    return new Response("Authentication required", { status: 401 });
  }
  
  // Your protected logic here
  
} catch (error) {
  console.error("Authentication error:", error);
  return new Response("Internal server error", { status: 500 });
}
```

Remember: **Security is not optional**. Every piece of user data must be properly protected and isolated by user ID.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iafhurtado) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
