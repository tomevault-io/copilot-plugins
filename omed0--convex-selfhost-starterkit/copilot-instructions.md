## convex-selfhost-starterkit

> Database triggers that automatically run code when data changes using convex-helpers


# Convex Database Triggers

Triggers automatically run code whenever data in a table changes. A library in the convex-helpers npm package allows you to attach trigger functions to your Convex database.

Triggers run within the same mutation that changes the data, so they run atomically with the data changing. Queries running in parallel will never see a state where the data has changed but the trigger didn't run.

## Setup

**Edit:** `convex/functions.ts`

```typescript
/* eslint-disable no-restricted-imports */
import {
  mutation as rawMutation,
  internalMutation as rawInternalMutation,
} from './_generated/server';
/* eslint-enable no-restricted-imports */
import { DataModel } from './_generated/dataModel';
import { Triggers } from 'convex-helpers/server/triggers';
import {
  customCtx,
  customMutation,
} from 'convex-helpers/server/customFunctions';

// Initialize triggers with table types from schema.ts
export const triggers = new Triggers<DataModel>();

// Import trigger registrations
import './triggers';

// Create wrappers that override ctx.db methods to run triggers
export const mutation = customMutation(rawMutation, customCtx(triggers.wrapDB));
export const internalMutation = customMutation(
  rawInternalMutation,
  customCtx(triggers.wrapDB)
);
```

**Edit:** `convex/triggers.ts`

```typescript
import { triggers } from './functions';

// Register a trigger for the "user" table
triggers.register('user', async (ctx, change) => {
  console.info('user changed', change);
});
```

## Basic Usage

### Register Triggers

```typescript
// Basic trigger registration
triggers.register('user', async (ctx, change) => {
  // change.operation: "insert" | "update" | "delete"
  // change.id: document ID
  // change.newDoc: new document (insert/update)
  // change.oldDoc: old document (update/delete)

  if (change.operation === 'delete') {
    console.info('user deleted', change.id);
  }
});
```

### Use Wrapped Mutations

```typescript
import { mutation } from './functions'; // NOT from "./_generated/server"

export const createUser = mutation({
  handler: async (ctx, args) => {
    // This automatically triggers registered functions
    await ctx.table('user').insert({ name: 'Alice' });
  },
});
```

## Common Use Cases

### Denormalizing Fields

```typescript
triggers.register('books', async (ctx, change) => {
  if (change.newDoc) {
    // Combine fields for full-text search
    const allFields = `${change.newDoc.title} ${change.newDoc.author} ${change.newDoc.summary}`;

    // Only update if changed to avoid infinite recursion
    if (change.newDoc.allFields !== allFields) {
      await ctx.table('books').getX(change.id).patch({ allFields });
    }
  }
});
```

### Data Validation

```typescript
triggers.register('user', async (ctx, change) => {
  if (change.newDoc) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(change.newDoc.email)) {
      throw new Error(`Invalid email: ${change.newDoc.email}`);
    }
  }
});
```

### Audit Logging

```typescript
triggers.register('teams', async (ctx, change) => {
  const user = await authComponent.safeGetAuthUser(ctx);
  await ctx.table('auditLog').insert({
    table: 'teams',
    operation: change.operation,
    documentId: change.id,
    userId: user?.tokenIdentifier,
    timestamp: Date.now(),
    changes: change,
  });
});
```

### Maintaining Aggregates

**Edit:** `convex/triggers.ts`

```typescript
import {
  aggregateUserFollowsByFollowing,
  aggregateUserFollowsByFollower,
} from './aggregates';

// Register aggregate triggers
triggers.register('follows', aggregateUserFollowsByFollowing.trigger());
triggers.register('follows', aggregateUserFollowsByFollower.trigger());
```

**CRITICAL:** For aggregates, ALWAYS use `aggregate.trigger()` - never update manually

### Authorization Rules

```typescript
triggers.register('messages', async (ctx, change) => {
  const identity = await authComponent.safeGetAuthUser(ctx);
  const userId = identity?.subject;

  const owner = change.oldDoc?.userId ?? change.newDoc?.userId;
  if (userId !== owner) {
    throw new Error(`User ${userId} cannot modify message owned by ${owner}`);
  }
});
```

### Asynchronous Processing

```typescript
const scheduled: Record<Id<'user'>, Id<'_scheduled_functions'>> = {};

triggers.register('user', async (ctx, change) => {
  // Cancel previous scheduled function
  if (scheduled[change.id]) {
    await ctx.scheduler.cancel(scheduled[change.id]);
  }

  // Schedule async processing
  scheduled[change.id] = await ctx.scheduler.runAfter(
    0,
    internal.users.processUserChange,
    { id: change.id, user: change.newDoc }
  );
});
```

## Best Practices

### 1. Prefer Explicit Functions

Instead of relying on "magic" triggers, consider explicit wrapper functions:

```typescript
// ✅ GOOD: Explicit and discoverable
async function createUser(ctx: MutationCtx, name: string) {
  const user = await ctx.table('user').insert({ name }).get();
  await ctx
    .table('userProfile')
    .insert({ userId: user._id, createdAt: Date.now() });
  console.info('user created', name);
  return user._id;
}

// ❌ AVOID: Hidden side effects
triggers.register('user', async (ctx, change) => {
  if (change.operation === 'insert') {
    await ctx.table('userProfile').insert({ userId: change.id });
  }
});
```

### 2. Always Use Wrappers

**CRITICAL:** Triggers only run when using wrapped mutations:

```typescript
// ✅ CORRECT: Import from functions.ts
import { mutation } from './functions';

// ❌ WRONG: Direct import bypasses triggers
import { mutation } from './_generated/server';
```

Triggers do NOT run when:

- Using unwrapped mutations
- Modifying data in Convex dashboard
- Using `npx convex import`
- Using streaming import

### 3. Avoid Infinite Recursion

Always check if update is needed:

```typescript
triggers.register('posts', async (ctx, change) => {
  if (change.newDoc) {
    const wordCount = change.newDoc.content.split(' ').length;

    // Only patch if different
    if (change.newDoc.wordCount !== wordCount) {
      await ctx.table('posts').getX(change.id).patch({ wordCount });
    }
  }
});
```

### 4. Handle Errors Carefully

**WARNING:** Caught errors don't roll back mutations:

```typescript
// ❌ BAD: Mutation succeeds even if trigger fails
export const updateMessage = mutation({
  handler: async (ctx, { id, body }) => {
    try {
      await ctx.table('messages').getX(id).patch({ body }); // Succeeds
    } catch (e) {
      console.error('failed'); // Trigger error caught
    }
    // Message is still updated!
  },
});

// ✅ GOOD: Let trigger errors bubble up
export const updateMessage = mutation({
  handler: async (ctx, { id, body }) => {
    await ctx.table('messages').getX(id).patch({ body }); // Trigger errors will fail mutation
  },
});
```

### 5. Consider Performance

- Triggers run synchronously in mutations
- Heavy processing should be scheduled async
- Batch operations when possible
- Be aware of write contention on single documents

## Real-World Example

**Edit:** `convex/triggers.ts` (from your codebase)

```typescript
triggers.register('characterWorks', aggregateCharacterWorks.trigger());
triggers.register('characterStars', aggregateCharacterStars.trigger());
triggers.register('characterSkills', aggregateCharacterSkills.trigger());

// Example of data validation trigger (still useful)
triggers.register('user', async (ctx, change) => {
  if (change.newDoc && !change.newDoc.email.includes('@')) {
    throw new Error('Invalid email');
  }
});
```

- Aggregate maintenance (`.trigger()` methods)
- Data validation
- Audit logging
- Custom business logic

## Trigger Change Object

The `change` parameter provides:

```typescript
interface Change<T> {
  operation: 'insert' | 'update' | 'delete';
  id: Id<T>;
  newDoc: Doc<T> | undefined; // Present for insert/update
  oldDoc: Doc<T> | undefined; // Present for update/delete
}
```

## Component Integration

Many Convex components provide trigger helpers:

```typescript
import { ShardedCounter } from '@convex-dev/sharded-counter';
const counter = new ShardedCounter(components.shardedCounter);

// Register component trigger
triggers.register('votes', counter.trigger('voteCount'));
```

## Summary

- Triggers run atomically with data changes
- Always use wrapped mutations from `functions.ts`
- Prefer explicit functions over implicit triggers
- Handle recursion and errors carefully
- Use for denormalization, validation, and custom business logic (cascade deletes now handled by Convex Ents)
- Integrate with aggregates via `.trigger()` methods

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Omed0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
