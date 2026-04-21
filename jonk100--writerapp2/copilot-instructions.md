## writerapp2

> This project uses **Astro as the primary framework** with:

# Astro Rules

## Core Principle

This project uses **Astro as the primary framework** with:

- Vanilla Astro components (`.astro`)
- Vanilla TypeScript
- Vanilla CSS

NO React, Vue, Svelte, or external UI frameworks.

---

## Hard Constraints

### 1. No JSX / React Patterns

❌ Forbidden:
- `useState`, `useEffect`, hooks
- JSX syntax inside `.ts` or `.astro`
- React-style event handlers (`onClick={() => ...}`)

✅ Required:
- Astro template syntax
- `<script>` for client-side behavior
- DOM event listeners

---

### 2. Astro Component Structure

```astro
---
/**
 * Component logic (server-side)
 */
---

<!-- HTML template -->

<script>
  // client-side JS only if needed
</script>

<style>
  /* scoped styles */
</style>
```

### 3. Client vs Server Seperation

- Frontmatter (---) is server-side only
- `<script>` tags are browser / client-side only

Never mix responsibilities

### 4. Data Fetching

- Prefer build-time `(getCollection)`
- Use SSR ONLY when necessary
- Avoid client-side fetching unless required for UX

### 5. Hydration

- Avoid over-hydration (only hydrate when necessary)

### 6. File Naming

- PascalCase.astro -> components
- camelCase.ts -> utils
- kebab-case.css -> styles

#### Anti-Patterns

- Avoid large, onolithic components
- Avoid business logic inside UI
- Avoid refetching data on client unnecessarily
- Avoid using React-style JSX patterns and hooks

#### Preferred Patterns

- Small, composable components with versatility
- Precomputed data
- Shared Utilities
- Type-safe schemas

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonk100) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
