## react-secure-notes

> SPA for managing notes securely.

# Context

SPA for managing notes securely.

## Tech Stack

- React + TypeScript
- Bundler -> Vite
- Tailwind CSS v4 (configured inline via Vite plugin — no separate config file)
- Component library: shadcn/ui (new-york style)
- Lexical for the rich text editor
- Redux -> UI state
- RTK Query -> server state
- React Router
- react-hook-form + Zod

## Architecture

We use bearer token auth, with refresh tokens (longer TTL) stored in httpOnly cookies and short-lived access tokens kept in Redux state.
Retries and redirect logic lives in `baseQueryWithReauth` (see `src/lib/auth.ts`).

`AuthGate` (`src/components/AuthGate.tsx`) wraps the whole app and redirects unauthenticated users to `/login`, except for routes listed in `PUBLIC_ROUTES` (`/login`, `/register`). Add any new public route there.

### State management

- Server data → RTK Query
- UI state → Redux slices

## Naming Conventions

| Category           | Convention                                        |
| ------------------ | ------------------------------------------------- |
| Feature components | PascalCase files + named functions                |
| UI primitives      | kebab-case files (button-group.tsx)               |
| Hook files         | camelCase (useMobile.ts, useStore.ts)             |
| Reducer files      | <name>.slice.ts / <name>.api.ts                   |
| Event handlers     | on prefix (onSubmit, onSave)                      |
| Redux actions      | camelCase verbs (selectNoteById, addNoteToRemove) |

## Folder Structure

```
src/
  App.tsx
  store.ts
  components/
    *.tsx              # feature-level smart components
    editor/            # Lexical subsystem (plugins, toolbar)
    ui/                # shadcn primitives (kebab-case filenames)
  hooks/
  lib/                 # utils, services...
  pages/               # route pages
  providers/           # context providers (react)
  reducers/            # slices, reducers (redux/RTK)
  schemas/             # form schemas (zod)
  selectors/           # selectors (redux)
  types/
```

## Recurring patterns

- `cn()` + `React.ComponentProps` spread — Every wrapping component accepts `className` and spreads `...props` through with `cn()`. Universal.
- CVA variants — Used in UI primitives for typed variant props.
- data-slot attributes — All UI primitives tag themselves for CSS parent selectors.
- Zod + react-hook-form + zodResolver — Identical boilerplate in every form.
- `FormInput` generic controller — Fully typed `useController` wrapper, reused across all form fields.
- `.unwrap()` on mutations — All RTK mutations unwrap for try/catch error handling.
- Toast on every async operation — sonner called after login, logout, create, save, and remove.
- Optimistic cache updates — createNote pushes to the getNotes cache immediately via onQueryStarted.

---
> Source: [gillesdeblock/react-secure-notes](https://github.com/gillesdeblock/react-secure-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
