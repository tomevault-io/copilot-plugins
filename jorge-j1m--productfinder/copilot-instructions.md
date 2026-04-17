## productfinder

> **CRITICAL**: This codebase follows a strict single-source-of-truth type system. All types MUST originate from database entity definitions.


# ARCHITECTURE PATTERNS

## DATABASE & TYPE SYSTEM

**CRITICAL**: This codebase follows a strict single-source-of-truth type system. All types MUST originate from database entity definitions.

### Entity Organization

Each database entity lives in `packages/database/src/entities/{entity}/` with this structure:

```
entities/stores/
  ├── schema.ts    # Drizzle table definition (SOURCE OF TRUTH)
  ├── id.ts        # Branded TypeID validators
  ├── types.ts     # Zod schemas derived from schema.ts
  └── index.ts     # Barrel exports
```

### Type Flow (MUST FOLLOW)

```
Drizzle Schema (schema.ts)
    ↓ via createSelectSchema/createInsertSchema
Zod Schemas (types.ts)
    ↓ via z.infer
TypeScript Types
    ↓ imported everywhere
Client & Server Code
```

**Rules:**

- ✅ ALL types derive from Drizzle schemas using `createSelectSchema`/`createInsertSchema`
- ✅ ONLY exception: Branded IDs get custom Zod refinements for type safety
- ❌ NEVER define custom types client-side or server-side that duplicate entity shapes
- ❌ NEVER use `as` to cast types
- ❌ NEVER use `as unknown as` or `any`

**Why**: A change in the database schema will automatically propagate as type errors throughout the codebase, preventing invisible bugs.

### Branded TypeID Pattern

All entity IDs use branded types with TypeID for compile-time safety and runtime validation:

```typescript
// In entities/{entity}/id.ts
import { fromString } from "typeid-js";
import { z } from "zod";
import { Brand } from "../../brand";

// 1. Define branded type
export type StoreId = Brand<string, "StoreId">;

// 2. Type predicate for runtime validation
export function isStoreId(id: unknown): id is StoreId {
  try {
    fromString(id as string, "store");
    return true;
  } catch {
    return false;
  }
}

// 3. Safe casting helper
export const asStoreId = (id: unknown): StoreId => {
  if (!isStoreId(id)) {
    throw new Error("Invalid StoreId");
  }
  return id as StoreId;
};

// 4. Zod schema for validation
export const storeIdSchema = z.custom<StoreId>((val) => isStoreId(val), {
  message: "Invalid StoreId format",
});
```

Then in `types.ts`:

```typescript
import { createInsertSchema, createSelectSchema } from "drizzle-zod";
import { stores } from "./schema";
import { storeIdSchema } from "./id";

// Override only the branded ID field
export const storeSchema = createSelectSchema(stores, {
  id: storeIdSchema,
  brandId: storeBrandIdSchema,
});

export const newStoreSchema = createInsertSchema(stores, {
  id: storeIdSchema.optional(),
  brandId: storeBrandIdSchema,
});

// Infer TypeScript types from Zod
export type Store = z.infer<typeof storeSchema>;
export type NewStore = z.infer<typeof newStoreSchema>;
```

### Using Drizzle ORM

- Always use Drizzle ORM for database operations
- Prefer the relational query API with `.query.{table}.findMany()` for relations
- Use branded types: `asStoreId()`, `asEmployeeId()`, etc.

## oRPC Procedures

**Pattern for custom entities** (stores, products, inventory, etc.):

```typescript
import { os } from "@orpc/server";
import { z } from "zod";
import { isStoreId, asStoreId, storeSchema, DB } from "@repo/database";

const osdb = os.$context<{ db: DB; requestId: string }>().errors({
  INTERNAL_SERVER_ERROR: { status: 500, message: "..." },
  NOT_FOUND: { status: 404, message: "..." },
  CONFLICT: { status: 409, message: "..." },
});

export const storesProcedures = {
  get: osdb
    .route({ method: "GET", path: "/stores/{id}", summary: "..." })
    .input(
      z.object({
        // Use string for OpenAPI compatibility, refine for validation
        id: z.string().refine(isStoreId, { message: "Invalid StoreId format" }),
      }),
    )
    .output(storeSchema)
    .handler(async ({ input, context, errors }) => {
      // Always refine to branded type in handler
      const id = asStoreId(input.id);

      const store = await context.db.query.stores.findFirst({
        where: (fields, { eq }) => eq(fields.id, id),
      });

      if (!store) throw errors.NOT_FOUND();
      return store;
    }),
};
```

**Important**:

- Route IDs are `z.string().refine(isXxxId, ...)` for OpenAPI generation
- Always use `asXxxId(input.id)` in the handler to get the branded type
- This pattern applies to ALL custom entities

**Exception**: For tables managed by better-auth (employees, employee_sessions, etc.), use better-auth's built-in functions and routes to maintain consistency with the authentication system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jorge-j1m) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
