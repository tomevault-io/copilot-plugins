## furqanagwan

> Core architectural principles for feature-first organization


# Vertical Slice Architecture - Core Principles

## Philosophy

**Features, not layers.** Organize code by business capability rather than technical role. Each feature should be self-contained and independently deployable.

---

## 1. Feature-First Organization

### Directory Structure

**RULE:** All feature-specific logic, UI, types, and data operations MUST live in `src/features/[feature-name]/`.

```
src/features/[feature-name]/
├── components/          # UI components used ONLY by this feature
│   ├── [Component].tsx
│   └── __tests__/
├── utils.ts            # Data transformation, sorting, formatting
├── utils.test.ts       # Utility tests (REQUIRED)
├── types.ts            # TypeScript interfaces/types
├── queries.ts          # Data fetching (server-side)
├── actions.ts          # Server mutations
├── hooks.ts            # Client-side hooks (if needed)
├── constants.ts        # Feature-specific constants
└── README.md           # Feature documentation
```

### Feature Completeness Checklist

| File            | Required When               |
| --------------- | --------------------------- |
| `types.ts`      | 2+ type definitions         |
| `utils.ts`      | Any data transformation     |
| `utils.test.ts` | `utils.ts` exists           |
| `constants.ts`  | Magic numbers/strings exist |
| `queries.ts`    | Server-side fetching needed |

### Shared Code Guidelines

**When to share code:**

- Component is used by **3+ distinct features** AND
- Component has no feature-specific logic AND
- Component is stable (unlikely to change per feature)

**Shared code location:**

```
src/shared/
├── components/         # Universal UI (Button, Input, Modal)
├── utils/             # Framework utilities (date formatting)
├── types/             # Domain-agnostic types
└── hooks/             # Common hooks (useDebounce)
```

**WHY 3+ features?** Premature abstraction creates maintenance debt. Wait for genuine reuse patterns to emerge.

---

## 2. Routing Layer (Framework-Specific)

### Next.js (`app/` directory)

**RULE:** Route files should be **<50 lines**. Extract logic to feature files if exceeding.

**RULE:** Avoid `"use client"` at route level. Create separate client components instead.

#### File Responsibilities

**`page.tsx`:**

- Fetch data via feature queries
- Render main feature component
- Pass data as props

**`layout.tsx`:**

- Define page structure and metadata
- Implement nested layouts

**`error.tsx`:**

- Handle feature-level errors
- Provide user-friendly error UI

**`loading.tsx`:**

- Show loading states during navigation
- Use Suspense for granular loading

#### Example

```typescript
// app/projects/page.tsx
import { ProjectsView } from '@/features/projects/components/ProjectsView';
import { getProjects } from '@/features/projects/queries';

export default async function ProjectsPage() {
  const projects = await getProjects();
  return <ProjectsView projects={projects} />;
}
```

**DO NOT:**

- Define complex UI in route files
- Perform data transformations inline
- Mix business logic with routing

### Framework Adaptations

- **Remix:** Use `routes/`, loaders, actions
- **SvelteKit:** Use `routes/`, `+page.server.ts`
- **Astro:** Use `pages/`, `getStaticProps`

---

## 3. Component Boundaries

### Server Components (Default in Next.js)

**USE FOR:**

- Data fetching with async/await
- Direct database/API access
- SEO-critical content
- No interactivity needed

```typescript
// Server Component
async function BlogPost({ slug }: { slug: string }) {
  const post = await getPost(slug);
  return <article>{post.content}</article>;
}
```

### Client Components (`'use client'`)

**USE FOR:**

- Event handlers (onClick, onChange)
- Browser APIs (localStorage, IntersectionObserver)
- React hooks (useState, useEffect)
- Third-party interactive libraries

```typescript
// Client Component
'use client';
import { useState } from 'react';

export function LikeButton({ postId }: { postId: string }) {
  const [likes, setLikes] = useState(0);
  return <button onClick={() => setLikes(likes + 1)}>❤️ {likes}</button>;
}
```

**RULE:** Push `'use client'` as deep as possible in the component tree to maximize server rendering benefits.

---

## 4. Data & State Management

### Static Content

Use framework-appropriate content management:

- **Next.js:** Content Collections, MDX
- **Astro:** Content Collections
- **General:** Markdown parsers with frontmatter

**RULE:** Import content APIs directly in Server Components or feature queries.

### Data Transformations

**ALWAYS** perform data operations in `features/[feature]/utils.ts`:

```typescript
// features/projects/utils.ts
export function sortProjectsByDate(projects: Project[]): Project[] {
  return [...projects].sort(
    (a, b) => new Date(b.date).getTime() - new Date(a.date).getTime(),
  );
}

export function groupProjectsByCategory(
  projects: Project[],
): Record<string, Project[]> {
  return projects.reduce(
    (acc, project) => {
      const category = project.category;
      acc[category] = [...(acc[category] || []), project];
      return acc;
    },
    {} as Record<string, Project[]>,
  );
}
```

### URL State

For filters, search, pagination, use URL-based state:

- **Next.js:** `nuqs`, `useSearchParams`
- **General:** URLSearchParams API

**WHY?** Shareable links, back button support, SEO benefits.

### Client State

For ephemeral UI state:

- **Simple:** `useState`, `useReducer`
- **Complex:** Zustand, Jotai, XState
- **Server Sync:** TanStack Query, SWR

---

## 5. Anti-Patterns to Avoid

### ❌ DON'T: Mix Concerns

```typescript
// BAD
export default async function Page() {
  const data = await fetch('/api/items');
  const items = data.json();
  const sorted = items.sort((a, b) => a.name.localeCompare(b.name));
  return <div>{sorted.map(item => <div>{item.name}</div>)}</div>;
}

// GOOD
export default async function Page() {
  const items = await getItems(); // queries.ts
  return <ItemsView items={items} />; // component
}
```

### ❌ DON'T: Overly Nested Directories

```
features/
  blog/
    components/
      post/
        header/
          title/  ❌ Too deep!
```

**RULE:** Maximum 2-3 levels of nesting.

### ❌ DON'T: Circular Dependencies

Feature A → Feature B → Feature A

**Fix:** Extract shared logic to `src/shared/` or reconsider boundaries.

### ❌ DON'T: Client Wrapping Server

```typescript
// BAD
'use client';
export function Wrapper({ children }) {
  return <div>{children}</div>; // children can't be Server Components!
}
```

**Fix:** Use composition - pass Server Components as props/children.

---

## Documentation Requirements

Each feature SHOULD include a README:

```markdown
# Feature Name

## Purpose

Brief description of what this feature does.

## Components

- `ComponentA`: Description
- `ComponentB`: Description

## API

- `getItems()`: Fetches items
- `createItem()`: Creates new item

## Dependencies

- External: stripe, react-query
- Internal: @/shared/utils

## Testing

Run: `npm test features/feature-name`
```

---

**Next:** See `02-validation-testing.md` for type safety and testing strategies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/furqanagwan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
