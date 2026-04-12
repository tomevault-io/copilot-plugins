## tallaby

> supabase


# Cursor Rules: Supabase + Drizzle Fetches

## Connection & RLS

- Use **postgres.js** + Drizzle. Point to **Supabase’s pooled connection** (Supavisor/pgBouncer). Disable prepared statements when using transaction pooling.
- Keep Drizzle **server-only**; never import the DB client in client components. Route all DB access via server actions / API routes.
- Enforce **Row-Level Security** in Postgres. When you must run RLS with a direct Postgres connection, set the **JWT claims** in the transaction to let `auth.uid()` work (see example wrapper below).
- Prefer **RLS policies** over ad-hoc “WHERE user_id = …” checks, and follow Supabase’s RLS best practices (least privilege, default-deny, explicit policies).

## Query building

- Build filters **declaratively** using Drizzle operators: `and`, `or`, `eq`, `ne`, `inArray`, `isNull`, `ilike`, `between`, etc. Use `$dynamic()` when many optional filters.
- **Search**: use `ilike(col, '%' + term + '%')`. For performance at scale, prefer `pg_trgm` or lowercased functional index.
- **Sorting**: never trust raw column names from input. Map a **string union** → **known columns**.
- **Selection**: explicitly project columns (or `with:` relations) to avoid leaking unintended data. Use relational queries for `with: { ... }`.

## Pagination

- **Offset/limit** for simple screens; ensure a **stable order** (e.g., `createdAt desc, id desc`) so paging is deterministic.
- For infinite lists, use **keyset (cursor) pagination** (tuple conditions on `(createdAt, id)`), which performs better at large offsets.

## Counting

- Run `count()` in a **separate query** with the **same WHERE** (no limit/offset). Heavy, but simple. Consider caching or materialized views if needed.

## Multi-tenant safety

- Always constrain by tenant (`storeId`) _and_ verify the authenticated user **owns** or **has access** to the tenant — enforced at **RLS** first, then optionally double-checked in code.

## Errors & Types

- Validate all inputs with **Zod**; create DTOs for outputs (or infer from Drizzle). Early-return on invalid state. Use typed errors (e.g., `AppError<'UNAUTHORIZED' | 'FORBIDDEN' | 'NOT_FOUND'>`).

## Performance

- Use pooled connections; **disable prepared statements** with postgres.js over Supavisor/pgBouncer: `postgres(DATABASE_URL, { prepare: false })`.
- Add indexes for common filters: `(store_id)`, `(type)`, `(status)`, `(created_at DESC, id DESC)`, and search helpers (`pg_trgm` on `name`).

## Migrations & RLS with Drizzle

- Keep schema and **RLS policies** in Drizzle migrations. Use `drizzle-orm/supabase` helpers (`authenticatedRole`, `authUsers`, `authUid`) to define policies, or link policies to existing Supabase tables.

---

## Drop-in Setup (server)

```ts
// src/db/index.ts
import "server-only";
import postgres from "postgres";
import { drizzle } from "drizzle-orm/postgres-js";

const DATABASE_URL = process.env.SUPABASE_DB_URL!; // pooled connection string
// Supavisor/pgBouncer: disable prepared statements
const client = postgres(DATABASE_URL, { max: 10, prepare: false });
export const db = drizzle({ client });
```

```ts
// src/db/rls.ts
// Wrap a transaction to run with the caller's Supabase JWT (so auth.uid() works in policies)
import { sql } from "drizzle-orm";
import type { PgTransaction } from "drizzle-orm/pg-core";
import { db } from "./index";

export interface SupabaseToken {
  sub?: string; // user id
  role?: string; // 'authenticated' | 'anon' | custom
  [k: string]: unknown;
}

// Run any Drizzle code inside an RLS-aware transaction
export async function withRls<T>(
  token: SupabaseToken | null,
  fn: (tx: PgTransaction<any, any, any>) => Promise<T>
): Promise<T> {
  return db.transaction(async (tx) => {
    if (token?.sub) {
      // Pass JWT claims to the DB session so auth.uid() / auth.jwt() work
      await tx.execute(sql`
        select set_config('request.jwt.claims', ${sql.raw(JSON.stringify(token))}, true);
        select set_config('request.jwt.claim.sub', ${token.sub ?? ""}, true);
        set local role ${sql.raw(token.role ?? "authenticated")};
      `);
    }
    try {
      return await fn(tx);
    } finally {
      await tx.execute(sql`
        select set_config('request.jwt.claims', null, true);
        select set_config('request.jwt.claim.sub', null, true);
        reset role;
      `);
    }
  });
}
```

---

## Zod inputs & safe sort helpers

```ts
// src/features/products/schemas.ts
import { z } from "zod";

export const SortKey = z.union([
  z.literal("created_at"),
  z.literal("name"),
  z.literal("price"),
]);

export const ProductQueryInput = z.object({
  name: z.string().trim().default(""),
  sort: SortKey.default("created_at"),
  direction: z.union([z.literal("asc"), z.literal("desc")]).default("desc"),
  offset: z.number().int().nonnegative().default(0),
  limit: z.number().int().positive().max(100).default(20),
  type: z.union([z.literal("single"), z.literal("bundle")]).default("single"),
  storeId: z.string().uuid(),
});

export type ProductQueryInput = z.infer<typeof ProductQueryInput>;
```

```ts
// src/features/products/sort-map.ts
import { asc, desc } from "drizzle-orm";
import { products } from "@/db/schema";

export function toOrderBy(
  sort: "created_at" | "name" | "price",
  direction: "asc" | "desc"
) {
  const col =
    sort === "name"
      ? products.name
      : sort === "price"
        ? products.price
        : products.createdAt;
  return direction === "asc" ? asc(col) : desc(col);
}
```

---

## Fetch templates

### Offset/limit query

```ts
// src/features/products/queries.ts
import { and, count, desc, eq, ilike, ne } from "drizzle-orm";
import { db } from "@/db";
import { withRls } from "@/db/rls";
import { products } from "@/db/schema";
import { ProductQueryInput } from "./schemas";
import { toOrderBy } from "./sort-map";

interface GetAuthCtx {
  (): Promise<{ token: import("@/db/rls").SupabaseToken | null }>;
}
export function makeProductsQueries(getAuthCtx: GetAuthCtx) {
  return {
    async getProducts(raw: unknown) {
      const parsed = ProductQueryInput.parse(raw);
      const { token } = await getAuthCtx();

      const base = [
        eq(products.storeId, parsed.storeId),
        eq(products.type, parsed.type),
        ne(products.status, "deleted"),
      ];
      if (parsed.name) {
        base.push(ilike(products.name, `%${parsed.name}%`));
      }

      const where = and(...base);
      return withRls(token, async (tx) => {
        const [rows, total] = await Promise.all([
          tx.query.products.findMany({
            where,
            with: { music: true },
            orderBy: toOrderBy(parsed.sort, parsed.direction),
            offset: parsed.offset,
            limit: parsed.limit,
          }),
          tx.select({ count: count() }).from(products).where(where),
        ]);

        return {
          data: rows,
          count: total[0]?.count ?? 0,
        };
      });
    },
  };
}
```

### Cursor (keyset) pagination

```ts
// src/features/products/queries.ts (continued)
import { and, eq, ilike, lt, or } from "drizzle-orm";
import { products } from "@/db/schema";

interface CursorInput {
  storeId: string;
  type?: "single" | "bundle";
  name?: string;
  limit?: number;
  cursor?: { createdAt: string; id: string } | null;
}

export function makeCursorProducts(getAuthCtx: GetAuthCtx) {
  return {
    async getProductsPage(input: CursorInput) {
      const { token } = await getAuthCtx();
      const limit = Math.min(Math.max(input.limit ?? 20, 1), 100);

      const filters = [
        eq(products.storeId, input.storeId),
        eq(products.type, input.type ?? "single"),
        ne(products.status, "deleted"),
      ];
      if (input.name) filters.push(ilike(products.name, `%${input.name}%`));

      return withRls(token, async (tx) => {
        const where = and(
          ...filters,
          input.cursor
            ? or(
                lt(products.createdAt, new Date(input.cursor.createdAt)),
                and(
                  eq(products.createdAt, new Date(input.cursor.createdAt)),
                  lt(products.id, input.cursor.id)
                )
              )
            : undefined
        );

        const rows = await tx.query.products.findMany({
          where,
          orderBy: [desc(products.createdAt), desc(products.id)],
          limit,
        });

        const next =
          rows.length === limit
            ? {
                createdAt: rows[rows.length - 1]!.createdAt,
                id: rows[rows.length - 1]!.id,
              }
            : null;

        return { data: rows, nextCursor: next };
      });
    },
  };
}
```

---

## Example RLS policy (Drizzle-defined)

```ts
// schema excerpt
import {
  pgPolicy,
  pgTable,
  uuid,
  text,
  timestamp,
  foreignKey,
} from "drizzle-orm/pg-core";
import { sql } from "drizzle-orm";
import { authenticatedRole, authUsers } from "drizzle-orm/supabase";

export const stores = pgTable(
  "stores",
  {
    id: uuid("id").primaryKey().notNull(),
    ownerId: uuid("owner_id").notNull(),
    name: text("name").notNull(),
  },
  (t) => [
    foreignKey({
      columns: [t.ownerId],
      foreignColumns: [authUsers.id],
      name: "stores_owner_fk",
    }).onDelete("cascade"),
    pgPolicy("stores_select_owner", {
      for: "select",
      to: authenticatedRole,
      using: sql`auth.uid() = ${t.ownerId}`,
    }),
  ]
);

export const products = pgTable(
  "products",
  {
    id: uuid("id").primaryKey().notNull(),
    storeId: uuid("store_id").notNull(),
    name: text("name").notNull(),
    status: text("status").notNull().default("active"),
    type: text("type").notNull(),
    price: text("price").notNull(),
    createdAt: timestamp("created_at", { withTimezone: true })
      .defaultNow()
      .notNull(),
  },
  (t) => [
    foreignKey({
      columns: [t.storeId],
      foreignColumns: [stores.id],
      name: "products_store_fk",
    }).onDelete("cascade"),
    pgPolicy("products_select_by_owner", {
      for: "select",
      to: authenticatedRole,
      using: sql`auth.uid() = (select s.owner_id from stores s where s.id = ${t.storeId})`,
    }),
  ]
);
```

---

## React Query hook (example)

```ts
// src/features/products/hooks.ts
import { useQuery } from "@tanstack/react-query";
import type { ProductQueryInput } from "./schemas";

export function useProducts(params: ProductQueryInput) {
  return useQuery({
    queryKey: ["products", params],
    queryFn: async () => {
      const res = await fetch("/api/products/search", {
        method: "POST",
        body: JSON.stringify(params),
        headers: { "content-type": "application/json" },
      });
      if (!res.ok) throw new Error("Failed to load products");
      return (await res.json()) as { data: unknown[]; count: number };
    },
    staleTime: 30_000,
    keepPreviousData: true,
  });
}
```

---

## Indexing notes

- **Sort**: index `(store_id, created_at desc, id desc)` or at least `(store_id, created_at desc)`.
- **Search**: enable `pg_trgm` and add `gin (name gin_trgm_ops)` or maintain a `name_normalized` column + btree index for prefix searches.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elsharkawy159)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/elsharkawy159)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
