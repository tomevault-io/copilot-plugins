## vue3-vite-project-template

> Here’s a **ready-to-paste rules file** for `/.windsurf/rules/code-style.md`.


Here’s a **ready-to-paste rules file** for `/.windsurf/rules/code-style.md`.
It’s scoped for your stack (Vue 3 + TS + Pinia/Colada + unplugin-vue-router + Supabase + shadcn-vue), theme-aware (dark/light), and enforces your store/typing patterns.

````md
# Code Style Guide — Vue 3 + TS + Supabase + shadcn-vue
activation: always

## 0) Scope & Non-negotiables
- Stack: Vue 3 + Vite + TypeScript (strict), Pinia (Options API stores), Pinia-Colada, unplugin-vue-router, Supabase, shadcn-vue + Tailwind, i18n (Client EN/TR; Admin EN).
- Never propose React/Next/JSX/TSX.
- File-based routing only (unplugin-vue-router). No manual route arrays.
- All UI must be **theme-aware** (dark/light). No hardcoded hex colors.
- Store actions that (re)fetch **must return** a **Query controller or Promise** so the **caller** controls loading state.
- Use **ConditionalContent** for all data UIs: `import { ConditionalContent } from '@/components/ui/conditional'`.

## 1) Files, Names & Imports
- Components: `PascalCase.vue` (e.g., `UserCard.vue`).
- Stores: `camelCase.ts` exporting `useXxxStore` (Options API).
- Composables: `useSomething.ts` (only when it improves reuse/clarity).
- Utility modules/types: `kebab-case.ts`.
- Import order: Node/3rd → `@/` aliases → relative. Use `import type` for types.

## 2) TypeScript
- **Strict** on. No `any`, no unchecked `JSON.parse`, avoid `@ts-ignore` (document rare exceptions).
- Prefer explicit return types on public functions, store actions, and composables.
- Narrow errors to `Error` instances; normalize unknowns via a helper (e.g., `toUserMessage(err)`).

## 3) Vue SFC Conventions
- Use `<script setup lang="ts">` + `<template>`. One component per file.
- Keep templates declarative; push heavy logic into script/composables.
- Reactivity: `ref` for primitives; `reactive` for grouped state; prefer `computed` over `watch`.
- When consuming Pinia state in components, use `storeToRefs(store)` before destructuring.

## 4) Routing (unplugin-vue-router)
- Pages live under `src/pages/**`. Group by section: `(client)/**` and `(admin)/**`.
- Use typed params & inferred names from the plugin.
- Add `meta.requiresAuth`, `meta.role`, `meta.section` as needed.
- Client: **EN/TR** (no hard-coded strings). Admin: **EN-only**.

## 5) State — Pinia (Options API only)
```ts
export const useThingStore = defineStore('thing', {
  state: () => ({ /* light, persistent state only */ }),
  getters: { /* pure derivations */ },
  actions: {
    /* async actions that return Promise or Colada controller */
  }
})
````

* No global `isLoading` flags for network calls in stores.
* Mutate state **only** inside store actions.

## 6) Data — Pinia-Colada (queries & mutations)

* Queries:

  ```ts
  const q = useQuery({
    key: () => ['things', filter.value],
    query: () => api.things.list(filter.value),
    staleTime: 300_000
  })
  ```
* Mutations:

  ```ts
  const { mutateAsync: save } = useMutation({
    mutation: api.things.upsert,
    onSuccess: (_res, dto) => {
      queryCache.invalidateQueries(['things'])
      if (dto.id) queryCache.invalidateQueries(['things', dto.id])
    }
  })
  ```
* **Refetch rule (enforced):** any action that (re)fetches **returns** a controller or Promise:

  ```ts
  // inside store
  refreshById(id: string) {
    return queryCache.refresh(['things', id]) // Promise the caller awaits
  }
  ```

## 7) Supabase — Typed Access & Centralized Types

* Never import `Tables<'...'>` all over the codebase. **Centralize types** in `@/types/index` and import from there in stores/components.
* `@/types/index` should:

  * Re-export core DB shapes from `database.types.ts` (e.g., `Tables`, `TablesInsert`, `TablesUpdate`).
  * Define **named domain types** per table (e.g., `Product`, `Variant`, `Category`).
  * Define **join types** (e.g., `ProductWithVariants`, `CategoryWithProductsAndVariants`).
  * Define **RPC types** (Args/Returns) for any used stored procedures.
* Wrap the Supabase client in `src/lib/*` typed helpers; keep component/store code free of raw SQL strings when practical.
* For inserts that should return rows, use `.insert(...).select().single()` (v2 behavior).

**Example: types/index.ts (pattern)**

```ts
// types/index.ts
import type { Database, Tables, TablesInsert, TablesUpdate} from './database.types'


// Domain aliases (reuse everywhere)
export type Product = Tables<'products'>;
export type ProductInsert = TablesInsert<'products'>;
export type ProductUpdate = TablesUpdate<'products'>;

// Joins
export type ProductWithVariants =
  Product & { product_variants: Tables<'product_variants'>[] };

export type CategoryWithProductsAndVariants =
  Tables<'product_categories'> & {
    products: (Product & { product_variants: Tables<'product_variants'>[] })[]
  };

// RPC example
export type UpdateStockQuantityArgs =
  Database['public']['Functions']['update_stock_quantity']['Args'];
export type UpdateStockQuantityReturn =
  Database['public']['Functions']['update_stock_quantity']['Returns'];
```

## 8) Store Pattern (reference implementation)

```ts
// src/stores/inventory.ts (Options API)
import { defineStore } from 'pinia'
import { useQuery, useMutation, useQueryCache } from '@pinia/colada'
import { supabase } from '@/lib/supabase'
import type { Product, ProductInsert, Variant, VariantInsert, CategoryWithProductsAndVariants, UpdateStockQuantityArgs } from '@/types'

export const useInventoryStore = defineStore('inventory', {
  state: () => ({ selectedCategoryId: null as string | null }),
  getters: {},
  actions: {
    // READS → return controllers (caller owns loading/error/success)
    useCategoriesProducts() {
      return useQuery({
        key: () => ['inventory', 'categories+products'],
        query: async () => {
          const { data, error } = await supabase
            .from('product_categories')
            .select('*, products(*, product_variants(*))')
          if (error) throw error
          return (data ?? []) as CategoryWithProductsAndVariants[]
        },
        staleTime: 300_000
      })
    },

    // WRITES → return Promises; invalidate precise keys
    async createProduct(input: ProductInsert): Promise<Product> {
      const { data, error } = await supabase.from('products').insert([input]).select().single()
      if (error) throw error
      const created = data as Product
      const cache = useQueryCache()
      cache.invalidateQueries(['inventory', 'categories+products'])
      cache.invalidateQueries(['products'])
      cache.invalidateQueries(['products', created.id])
      return created
    },

    async createVariants(inputs: VariantInsert[]): Promise<Variant[]> {
      if (!inputs?.length) return []
      const { data, error } = await supabase.from('product_variants').insert(inputs).select()
      if (error) throw error
      const variants = (data ?? []) as Variant[]
      const cache = useQueryCache()
      cache.invalidateQueries(['inventory', 'categories+products'])
      variants.forEach(v => cache.invalidateQueries(['products', v.product_id]))
      return variants
    },

    async updateStockViaRpc(args: UpdateStockQuantityArgs) {
      const { data, error } = await supabase.rpc('update_stock_quantity', args)
      if (error) throw error
      const cache = useQueryCache()
      cache.invalidateQueries(['inventory', 'categories+products'])
      if ('variant_id' in args) cache.invalidateQueries(['variants', args.variant_id])
      return data
    }
  }
})
```

## 9) UI/UX — shadcn-vue + Tailwind (Theme-aware)

* Use shadcn-vue primitives; customize via props/variants, not ad-hoc CSS.
* Use tokens/CSS vars only: `bg-background`, `text-foreground`, `text-muted-foreground`, `border-border`, `ring-ring`, etc.
* Prefer **light borders/dividers** over heavy shadows; maintain a clear visual hierarchy.
* Keep accessible focus states; never remove outlines without a visible replacement.

## 10) UI State — ConditionalContent (required)

* Always wrap data views:

  ```vue
  <ConditionalContent
    :is-loading="q.isLoading"
    :has-error="q.status==='error'"
    :error="q.error"
    :retry="q.refresh"
    :is-empty="Array.isArray(q.data) ? q.data.length===0 : !q.data"
    empty-title="No items"
    empty-message="Create your first item."
    empty-icon="package"
  >
    <ItemList :items="q.data!" />
  </ConditionalContent>
  ```
* Multi-query pages: combine `isLoading` with `||`, `error` with `??`, and `retry` calls each controller’s `refresh()`.
* Icon strings allowed: `database|search|file|users|package|calendar|bell|settings|inbox`.

## 11) Composables

* Create only when reused ≥2 places or they materially simplify complex components.
* Input/output must be typed. Composables must not leak raw Supabase client unless they are domain lib wrappers.

## 12) Errors & Results

* Throw `Error` objects (never raw strings). Map to user-facing text in UI (i18n on Client).
* Components render errors exclusively via `ConditionalContent`’s error slot; provide `retry` where safe.

## 13) i18n

* Client: EN & TR; no hard-coded user-facing text. Admin: English only.
* Keep locale keys consistent and descriptive. Wrap date/number formatting in i18n-aware utils.

## 14) Performance

* Code-split large views with `defineAsyncComponent`.
* Tune `staleTime` realistically; avoid chatty re-fetch loops.
* Prefer DB views/RPCs for heavy joins; index frequently queried columns.

## 15) Testing & Linting

* ESLint: `eslint-plugin-vue`, `@typescript-eslint`. Prettier aligned with **100** column width.
* `vue-tsc` in CI for type checks.
* Unit test stores and `src/lib/*`; smoke test critical pages. Mock Supabase in tests.

## 16) PR Hygiene

* Small, single-concern PRs.
* Checklist: centralized types used; queries/mutations follow §6; `ConditionalContent` wraps data UIs; i18n keys added; theme tokens only; tests updated.

## 17) Bans (enforced)

* React/Next/JSX/TSX, manual vue-router arrays, blanket cache invalidation, global store loading flags, heavy shadows, hardcoded colors, duplicative composables, `any`, `@ts-ignore` (unreviewed).

```

If you want, I can also generate a tiny **ESLint+Prettier** config (import order, no `any`, 100-col, Vue SFC rules) and a `src/lib/api` skeleton to enforce these conventions automatically.
::contentReference[oaicite:0]{index=0}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PraiseSinkamba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
