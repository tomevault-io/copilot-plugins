## react-router-starter

> Storybook setup and patterns for React Router apps


# Storybook Cursor Rules

## Setup (todo-app)

- **Framework:** `@storybook/react-vite` with Vite config from app root
- **Stories:** `../app/**/*.stories.@(ts|tsx)`
- **Addons:** `@storybook/addon-essentials`, `@storybook/addon-links`
- **Config:** `main.ts` uses `viteFinal` to resolve `vite.config.ts` for path aliases and Tailwind

## React Router in Storybook

Components that use `Link`, `useNavigate`, `useFetcher`, or `useHref` need router context. Use a global decorator in `preview.tsx`:

```tsx
import type { Preview } from '@storybook/react';
import type { ComponentType } from 'react';
import { createMemoryRouter, RouterProvider } from 'react-router-dom';

const withRouter = (Story: ComponentType) => {
  const router = createMemoryRouter(
    [{ path: '/', element: <Story /> }],
    { initialEntries: ['/'] }
  );
  return <RouterProvider router={router} />;
};

const preview: Preview = {
  decorators: [withRouter, withTodoProvider], // add app providers as needed
  // ...
};
```

## Patterns

- **Single route:** One route with `element: <Story />` and `initialEntries: ['/']` (or desired path)
- **App providers:** Add decorators for context providers (e.g. TodoProvider) so stories match app behavior
- **Global styles:** Import app `globals.css` in `preview.tsx` so Tailwind and theme apply

## Scripts

- **Dev:** `npm run storybook` (port 6006)
- **Build:** `npm run build-storybook` (static output for deployment)

## Best Practices

- Keep stories colocated with components (`component.stories.tsx`)
- Use args and controls for props; use decorators for router and providers
- Prefer one default export with a descriptive title; use named exports for variants

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lambda-curry)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lambda-curry)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
