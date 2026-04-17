## flashycardy

> This project uses **Clerk** for all authentication. **CRITICAL**: Users can ONLY access their own data. All database operations must enforce user isolation.

# Clerk Authentication & Data Security Rules

This project uses **Clerk** for all authentication. **CRITICAL**: Users can ONLY access their own data. All database operations must enforce user isolation.

## Authentication Setup

### Required Imports
```typescript
import { auth, currentUser } from "@clerk/nextjs/server";
import { redirect } from "next/navigation";
```

### User Authentication Patterns

#### 1. Server Components - Get Current User
```typescript
// ✅ Correct - Get authenticated user in server components
import { auth } from "@clerk/nextjs/server";

export default async function MyPage() {
  const { userId } = await auth();
  
  if (!userId) {
    redirect("/"); // Redirect to homepage where buttons are located
  }
  
  // Use userId for data queries
  const userDecks = await db.select()
    .from(decksTable)
    .where(eq(decksTable.userId, userId));
}
```

#### 2. Server Actions - Validate User
```typescript
// ✅ Correct - Validate user in server actions
import { auth } from "@clerk/nextjs/server";

export async function createDeck(formData: FormData) {
  const { userId } = await auth();
  
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  // Always include userId when creating data
  await db.insert(decksTable).values({
    name: formData.get("name") as string,
    userId: userId, // CRITICAL: Always set userId
  });
}
```

#### 3. API Routes - Protect Endpoints
```typescript
// ✅ Correct - Protect API routes
import { auth } from "@clerk/nextjs/server";

export async function GET() {
  const { userId } = await auth();
  
  if (!userId) {
    return Response.json({ error: "Unauthorized" }, { status: 401 });
  }
  
  // Query only user's data
  const decks = await db.select()
    .from(decksTable)
    .where(eq(decksTable.userId, userId));
    
  return Response.json(decks);
}
```

## Data Access Control Rules

### 1. ALWAYS Filter by User ID
```typescript
// ✅ Correct - Always filter by userId
const userDecks = await db.select()
  .from(decksTable)
  .where(eq(decksTable.userId, userId));

// ❌ WRONG - Never query without user filter
const allDecks = await db.select().from(decksTable);
```

### 2. Validate Ownership Before Operations
```typescript
// ✅ Correct - Verify ownership before updates/deletes
export async function updateDeck(deckId: number, data: any) {
  const { userId } = await auth();
  if (!userId) throw new Error("Unauthorized");
  
  // First verify the deck belongs to the user
  const deck = await db.select()
    .from(decksTable)
    .where(and(eq(decksTable.id, deckId), eq(decksTable.userId, userId)))
    .limit(1);
    
  if (deck.length === 0) {
    throw new Error("Deck not found or access denied");
  }
  
  // Then perform the update
  await db.update(decksTable)
    .set(data)
    .where(eq(decksTable.id, deckId));
}
```

### 3. Use Transactions for Multi-Table Operations
```typescript
// ✅ Correct - Use transactions with user validation
export async function createDeckWithCards(deckData: any, cardsData: any[]) {
  const { userId } = await auth();
  if (!userId) throw new Error("Unauthorized");
  
  return await db.transaction(async (tx) => {
    // Create deck with userId
    const [deck] = await tx.insert(decksTable).values({
      ...deckData,
      userId: userId,
    }).returning();
    
    // Create cards with deckId (automatically user-owned via deck)
    const cards = await tx.insert(cardsTable).values(
      cardsData.map(card => ({
        ...card,
        deckId: deck.id,
      }))
    ).returning();
    
    return { deck, cards };
  });
}
```

## Database Schema Security

### User Isolation Fields
- **decksTable**: `userId` field ensures deck ownership
- **cardsTable**: Inherits user ownership through `deckId` → `decksTable.userId`

### Required Query Patterns
```typescript
// ✅ Correct - Always include user filter
import { eq, and } from "drizzle-orm";

// Get user's decks
const userDecks = await db.select()
  .from(decksTable)
  .where(eq(decksTable.userId, userId));

// Get user's cards through deck relationship
const userCards = await db.select()
  .from(cardsTable)
  .innerJoin(decksTable, eq(cardsTable.deckId, decksTable.id))
  .where(eq(decksTable.userId, userId));

// Verify deck ownership before card operations
const deckWithCards = await db.select()
  .from(decksTable)
  .leftJoin(cardsTable, eq(decksTable.id, cardsTable.deckId))
  .where(and(
    eq(decksTable.id, deckId),
    eq(decksTable.userId, userId)
  ));
```

## Client-Side Protection

### Protected Routes
```typescript
// ✅ Correct - Use Clerk components for client-side protection
import { SignedIn, SignedOut } from "@clerk/nextjs";
import { redirect } from "next/navigation";

export default function ProtectedPage() {
  return (
    <>
      <SignedIn>
        {/* Protected content */}
        <UserDashboard />
      </SignedIn>
      <SignedOut>
        {redirect("/")}
      </SignedOut>
    </>
  );
}
```

### User Information
```typescript
// ✅ Correct - Get user info on client
import { useUser } from "@clerk/nextjs";

export function UserProfile() {
  const { user, isLoaded } = useUser();
  
  if (!isLoaded) return <div>Loading...</div>;
  if (!user) return <div>Please sign in to continue</div>;
  
  return <div>Welcome, {user.firstName}!</div>;
}
```

## Security Checklist

### Before Every Database Operation:
1. ✅ Get `userId` from `auth()`
2. ✅ Verify `userId` exists (redirect if not)
3. ✅ Include `userId` filter in ALL queries
4. ✅ Validate ownership before updates/deletes
5. ✅ Use transactions for multi-table operations
6. ✅ Never expose other users' data

### Common Security Mistakes to Avoid:
- ❌ Querying without user filter
- ❌ Trusting client-side user ID
- ❌ Skipping ownership validation
- ❌ Exposing user data in API responses
- ❌ Using raw SQL without user constraints

## Error Handling
```typescript
// ✅ Correct - Proper error handling
export async function getUserDecks() {
  try {
    const { userId } = await auth();
    
    if (!userId) {
      throw new Error("Authentication required");
    }
    
    const decks = await db.select()
      .from(decksTable)
      .where(eq(decksTable.userId, userId));
      
    return { success: true, data: decks };
  } catch (error) {
    console.error("Database error:", error);
    return { success: false, error: "Failed to fetch decks" };
  }
}
```

## Environment Variables
Ensure these Clerk environment variables are set:
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
- `CLERK_SECRET_KEY`
- `NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL`
- `NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlanKing82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
