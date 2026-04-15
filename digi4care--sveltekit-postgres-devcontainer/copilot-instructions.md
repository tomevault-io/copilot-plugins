## sveltekit-postgres-devcontainer

> Description


# General Code Style & Formatting

- Follow the Airbnb Style Guide for code formatting.
- Use PascalCase for Svelte 5 component file names (e.g., UserCard.svelte, not
  user-card.svelte).
- Prefer named exports for components.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading,
  hasError).
- Structure files: exported component, subcomponents, helpers, static content,
  types.
- Follow Svelte 5's official documentation and best practices for all
  development.

## Project Structure & Architecture

- Follow SvelteKit patterns and file-based routing
- Correctly determine when to use server vs. client components in SvelteKit
- Always use Svelte 5 syntax with runes for reactivity
- Leverage Svelte 5's new features like snippets and enhanced bindings
- Use the `@runed/svelte` package for enhanced reactivity patterns

## Naming Conventions

- Use lowercase with dashes for directories (e.g., components/auth-wizard).
- Favor named exports for components.

## TypeScript Best Practices

- Use TypeScript for all code; prefer interfaces over types.
- Avoid any and enums; use explicit types and maps instead.
- Use functional components with TypeScript interfaces.
- Enable strict mode in TypeScript for better type safety.

## Styling & UI

- Use Tailwind 4 CSS for styling.
- Use ShadcnSvelte UI for components, use only the Svelte 5 version.
- Ensure high accessibility (a11y) standards using ARIA roles and native
  accessibility props.

## Data Fetching & Forms

- Use SvelteKit's `load` functions for server-side data fetching
- For client-side data fetching, use `fetch` with `$effect` for reactive data
  loading
- Use `@runed/svelte` form utilities for form handling with Svelte 5
- Use Zod for schema validation with Svelte 5 runes
- Prefer `@attach` over `:use` for better type safety and Svelte 5 compatibility
- Leverage runes for form state management and validation

```svelte
<script>
    import { $state, $derived } from "svelte";
    import { createForm } from "@runed/svelte";

    const { form, handleSubmit } = createForm({
        initialValues: { email: "", password: "" },
        onSubmit: (values) => console.log(values)
    });
</script>
```

## State Management & Logic with Svelte 5

### Reactivity with Runes

- Use Svelte 5 runes for all reactive state management:
    - `$state` for reactive state
    - `$derived` for computed values
    - `$effect` for side effects
    - `$props` for component props
    - `$bindable` for two-way binding props
    - `$inspect` for debugging reactive values

### Context API

Context provides a type-safe way to share data across the component tree without
prop drilling. Use it for:

- Authentication state
- Theme preferences
- Localization settings
- Feature flags
- Any global application state

#### Basic Usage

```svelte
<!-- ContextProvider.svelte -->
<script>
  import { setContext } from 'svelte';
  import { writable } from 'svelte/store';

  // Create and set context
  const theme = writable('light');
  setContext('theme', theme);
</script>

<slot />

<!-- ContextConsumer.svelte -->
<script>
  import { getContext } from 'svelte';

  // Get context value
  const theme = getContext('theme');
</script>

<div class="{$theme}">
  Current theme: {$theme}
</div>
```

### Using with `@runed/svelte`

The `@runed/svelte` package enhances context with additional features:

#### Type-Safe Context

```typescript
// context.ts
import { createContext } from '@runed/svelte';

export const ThemeContext = createContext<{
  theme: 'light' | 'dark';
  toggle: () => void;
}>('theme');

// Provider.svelte
<script>
  import { ThemeContext } from './context';
  import { $state } from 'svelte';

  const theme = $state('light');

  function toggle() {
    theme.update(t => t === 'light' ? 'dark' : 'light');
  }

  ThemeContext.set({ theme, toggle });
</script>

<slot />

// Consumer.svelte
<script>
  import { ThemeContext } from './context';

  const { theme, toggle } = ThemeContext.get();
</script>

<button on:click={toggle}>
  Toggle Theme (Current: {$theme})
</button>
```

### Best Practices

1. **Type Safety**: Always type your context values
2. **Default Values**: Provide sensible defaults for better DX
3. **Documentation**: Document context structure and usage
4. **Performance**: Use `$derived` for expensive computations
5. **Testing**: Test context providers and consumers in isolation

### When to Use Context vs. Props vs. Stores

- **Props**: Parent-to-child communication
- **Context**: Deeply nested component trees
- **Stores**: Global application state that needs to be accessed from anywhere
- **Runes**: Component-local state and fine-grained reactivity

### Advanced Patterns

- **Composition**: Combine multiple contexts
- **Scoping**: Create scoped contexts for specific features
- **Middleware**: Add logging or validation to context updates
- **Persistence**: Automatically persist context to localStorage

## Backend & Database

Use Drizzle or SQLite3 for database access. Depends what is configured or asked.
If unclear ask first, never assume.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/digi4care) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
