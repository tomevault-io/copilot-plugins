## openstock

> Guidelines for AI coding agents working in the OpenStock codebase.

# AGENTS.md

Guidelines for AI coding agents working in the OpenStock codebase.

## Project Overview

Self-hosted inventory and stock management system:
- **Framework**: Nuxt 4 with Vue 3 Composition API
- **Database**: Cloudflare D1 (SQLite) with Drizzle ORM
- **Styling**: TailwindCSS
- **Hosting**: Cloudflare Pages via NuxtHub

## Build/Lint/Test Commands

```bash
# Development
npm run dev              # Start dev server on http://localhost:3000

# Build
npm run build            # Production build
npm run preview          # Preview production build locally

# Database
npm run db:generate      # Generate migrations from schema changes
npm run db:populate      # Populate database with sample data
npm run db:export        # Export database
npm run db:sync          # Export and push to remote

# Deployment
npm run deploy:cf        # Build and deploy to Cloudflare Pages
npm run postinstall      # nuxt prepare (run after npm install)
```

**Note**: No test framework or linter is currently configured.

## Project Structure

```
├── app/                    # Frontend (Nuxt 4 app directory)
│   ├── components/         # Vue components (ui/, charts/)
│   ├── composables/        # Vue composables (useAuth, useToast, useSettings)
│   ├── layouts/            # Page layouts
│   ├── pages/              # File-based routing
│   └── middleware/         # Route middleware
├── server/                 # Backend (Nitro)
│   ├── api/                # API endpoints (file-based routing)
│   ├── database/           # Drizzle schema (schema.ts)
│   ├── middleware/         # Server middleware (auth.ts)
│   └── utils/              # Server utilities (db.ts, id.ts)
├── migrations/             # SQL migration files
└── nuxt.config.ts          # Nuxt configuration
```

## Code Style Guidelines

### Imports

```typescript
// 1. External packages first
import { eq, desc } from 'drizzle-orm';

// 2. Server schema imports using the ~ alias
import { products } from '~~/server/database/schema';
import type { Product, Category } from '~~/server/database/schema';

// Nuxt auto-imports (no explicit import needed): useDB, tables, defineEventHandler, ref, reactive, computed, useFetch
```

### TypeScript

- Use TypeScript for all new files
- Prefer type inference; explicit types for API responses and public interfaces
- Use `interface` for object shapes, `type` for unions
- Schema types from Drizzle: `export type Product = typeof products.$inferSelect;`

### Vue Components

```vue
<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
});

const emit = defineEmits<{
  (e: 'update', value: string): void;
}>();
</script>
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase.vue | `AppHeader.vue` |
| Composables | camelCase.ts, use-prefix | `useAuth.ts` |
| API routes | kebab-case | `index.get.ts`, `[id].put.ts` |
| Variables/functions | camelCase | `const products = ref([])` |
| Database tables | camelCase (plural) | `products`, `stockMovements` |
| IDs | prefix_uuid | `prod_abc123`, `var_def456` |

### API Endpoints

**File naming:** `index.get.ts`, `index.post.ts`, `[id].get.ts`, `[id].put.ts`, `[id].delete.ts`

```typescript
export default defineEventHandler(async (event) => {
  const db = useDB();
  const id = getRouterParam(event, 'id');
  const body = await readBody(event);

  if (!id) {
    throw createError({ statusCode: 400, message: 'ID is required' });
  }

  const item = await db.query.products.findFirst({
    where: eq(tables.products.id, id),
  });

  if (!item) {
    throw createError({ statusCode: 404, message: 'Item not found' });
  }

  return item;
});
```

### Error Handling

```typescript
// API errors
throw createError({ statusCode: 400, message: 'Invalid input' });
throw createError({ statusCode: 401, message: 'Authentication required' });
throw createError({ statusCode: 403, message: 'Access denied' });
throw createError({ statusCode: 404, message: 'Resource not found' });

// Vue components
try {
  await $fetch('/api/products', { method: 'POST', body: payload });
  toast.success('Success', 'Product created');
} catch (error) {
  console.error(error);
  toast.error('Error', 'Failed to save product');
} finally {
  isSubmitting.value = false;
}
```

### Database Operations

```typescript
const db = useDB();

// Query with relations
const products = await db.query.products.findMany({
  orderBy: [desc(tables.products.createdAt)],
  with: { category: true, supplier: true, variants: true },
});

// Insert
const id = generateId('prod');
await db.insert(tables.products).values({ id, name: 'Product' });

// Update
await db.update(tables.products)
  .set({ stockQuantity: newStock, updatedAt: new Date() })
  .where(eq(tables.products.id, id));
```

### Styling (TailwindCSS)

- Use Tailwind utility classes directly in templates
- Group utilities: layout → spacing → typography → colors → effects
- Semantic colors: `primary-600`, `gray-100`, `red-50`
- Responsive prefixes: `sm:`, `md:`, `lg:`

```html
<button class="inline-flex items-center justify-center rounded-md bg-primary-600 px-4 py-2 text-sm font-medium text-white hover:bg-primary-700">
  Save
</button>
```

## Important Notes

1. **Nuxt auto-imports**: No explicit imports for `ref`, `reactive`, `computed`, `useFetch`, `defineEventHandler`, `useDB`, `tables`

2. **Database migrations**: After modifying `server/database/schema.ts`, run `npm run db:generate`

3. **ID generation**: Use `generateId('prefix')` from `server/utils/id.ts`

4. **Toast notifications**: Use `const toast = useToast()` for user feedback

5. **Type imports**: Use `import type { }` for type-only imports

6. **Authentication**: Session via `nuxt-auth-utils`. Server middleware enforces role-based access (admin, member, viewer)

---
> Source: [florianjs/openstock](https://github.com/florianjs/openstock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
