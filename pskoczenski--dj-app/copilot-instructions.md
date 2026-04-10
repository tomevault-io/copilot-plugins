## dj-app

> This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

---

## Stack

- **Next.js 16** (App Router) / **React 19**
- **Supabase** (Postgres, Auth, Storage) via `@supabase/ssr` + `@supabase/supabase-js`
- **Tailwind CSS v4** with custom design tokens
- **shadcn/ui** (v4) for component primitives
- **Jest 30** + **React Testing Library** for tests
- **TypeScript** (strict)

---

## Extended context

This file is the **compact engineering contract**. For **product orientation**, **domain summaries**, **documentation / planning workflow for agents**, **TODO pointers**, and **recent work**, read **`docs/project-context-for-agents.md`** first on larger or unfamiliar tasks.

---

## Architecture & data flow

All data flows through a consistent pattern. Do not skip layers.

```
Supabase client → lib/services/*.ts → hooks/*.ts → components/*.tsx
```

- **Services** (`lib/services/`) contain all Supabase queries. Components and hooks never call Supabase directly.
- **Hooks** (`hooks/`) wrap service calls with React state (loading, error, data). They handle polling, optimistic updates, and refetch logic.
- **Components** consume hooks and render UI. They never import from `@supabase/supabase-js` or `lib/supabase/` directly.

---

## Supabase conventions

### Client creation

- **Browser (client components)**: use `createBrowserClient()` from `lib/supabase/client.ts`.
- **Server (server components, route handlers, server actions)**: use `createServerClient()` from `lib/supabase/server.ts`.
- **Middleware**: use the middleware helper from `lib/supabase/middleware.ts`.
- **Never** import `createClient` directly from `@supabase/supabase-js` — always use the project wrappers.

### RLS

- RLS is enabled on every table. Never bypass it.
- Never use the `service_role` key in client-side code.
- All queries run as the authenticated user via the anon key + session. Trust RLS to enforce access control.

### Soft deletes

The following tables use soft delete via `deleted_at timestamptz`:
- `profiles`, `events`, `mixes`, `messages`, `comments`

**Never use hard DELETE** on these tables. Set `deleted_at = now()` instead. All SELECT queries must filter `deleted_at IS NULL` (RLS policies enforce this for most tables, but verify in service code).

Hard DELETE is used only for:
- `follows` (unfollow)
- `comment_likes` (unlike)
- `conversation_participants` (removal from group thread)

### Genres (master table)

Profiles, events, and mixes store **`genre_ids uuid[]`** (FK to the **`genres`** reference table). The UI still works with string labels; **`genresService`** resolves labels to IDs on write and **`hydrateGenreLabels`** attaches a **`genres: string[]`** array on reads. Do not call Supabase RPCs for genres—there is no `genre_tags` / `upsert_genre_tags`.

### Storage buckets

Three public buckets: `profile-images`, `event-flyers`, `mix-covers`. Path conventions:
- Profile image: `<user_id>/avatar.<ext>`
- Event flyer: `<event_id>/flyer.<ext>` (or `<user_id>/draft-flyer.<ext>` before the event exists)
- Mix cover: `<mix_id>/cover.<ext>`

Always upload with `upsert: true`. Client-side validation: JPEG/PNG/WebP, max 5 MB.

---

## Routing & file conventions

### App Router structure

```
app/
  (auth)/         — login, signup (unauthenticated)
  (main)/         — authenticated shell (home, events, mixes, search, dj profiles, messages)
```

- `(auth)` and `(main)` are route groups with separate layouts.
- Pages are **server components by default**. Add `"use client"` only when the component uses hooks, event handlers, or browser APIs.
- Layouts handle auth checks and shared chrome (navbar, mobile bars).

### Component organization

```
components/
  ui/             — shadcn/ui primitives (do not edit directly unless customizing)
  shared/         — reusable app-wide (EmptyState, LoadingSpinner, etc.)
  layout/         — navbar, mobile bars, avatar dropdown
  events/         — EventCard, EventForm, EventCalendar, etc.
  mixes/          — MixEmbed, MixCard, MixForm, etc.
  messages/       — QuickMessageDialog, ComposeBar, etc.
  comments/       — CommentsModal, CommentList, etc.
  profile/        — ProfileHeader, FollowButton, SocialLinks
  forms/          — GenreSelect, ImageUpload
```

Place new components in the appropriate feature directory. If a component is used across multiple features, put it in `shared/`.

### Naming

- Files: kebab-case (`event-card.tsx`, `use-comments.ts`)
- Components: PascalCase (`EventCard`, `CommentsModal`)
- Services: camelCase object with methods (`eventsService.getAll()`)
- Hooks: camelCase with `use` prefix (`useComments`, `useCalendarEvents`)
- Types: PascalCase (`CalendarEvent`, `CommentWithAuthor`)

---

## Styling

### Tailwind v4 + design tokens

The app uses Tailwind CSS v4 with custom design tokens defined in the CSS layer. **Do not hardcode hex colors, pixel values, or font sizes.** Use the token classes defined in the design system.

Design specs live in `docs/design_specs/`. Reference them for:
- `01` — Brand colors, surface colors, accent palette
- `02` — Typography scale and font families
- `03` — Border radii, shadows, focus rings
- `05` — Component-level styles (buttons, inputs, cards, dialogs)
- `06` — Layout grid, spacing scale, breakpoints
- `07` — Motion/animation conventions, `prefers-reduced-motion`

### shadcn/ui

Use shadcn components (`Dialog`, `Button`, `Input`, `Textarea`, `Select`, etc.) as the base. Do not build custom modals, dropdowns, or form controls from scratch unless shadcn doesn't cover the use case.

### Dark mode

The app uses `next-themes`. Ensure all UI respects both light and dark mode via Tailwind's `dark:` variants and the design token CSS variables.

---

## Testing

### Setup

- **Framework**: Jest 30 with `jest-environment-jsdom`
- **Libraries**: `@testing-library/react`, `@testing-library/jest-dom`, `@testing-library/user-event`
- **Config**: `jest.config.ts` and `jest.setup.ts` in project root

### File structure

Tests mirror the source tree inside `__tests__/`:
```
__tests__/
  lib/services/events.test.ts
  hooks/use-comments.test.ts
  components/comments/CommentsModal.test.tsx
```

### Patterns

- **Services**: mock the Supabase client and its query chain. Assert that the correct table, filters, ordering, and range are called.
- **Hooks**: use `renderHook` from `@testing-library/react`. Mock the underlying service. Test loading → success and loading → error paths. For polling hooks, use `jest.useFakeTimers()`.
- **Components**: render with mocked hooks/props. Test user interactions (click, type, submit) and visible output. Use `screen.getByRole`, `screen.getByText`, etc. — prefer accessible queries.
- **Do not** make real Supabase calls in tests. All external dependencies are mocked.

---

## Common mistakes to avoid

- **Don't query Supabase in components.** Always go through a service function, then a hook.
- **Don't hard DELETE soft-delete tables.** Use `deleted_at = now()`.
- **Don't hardcode colors or spacing.** Use Tailwind classes and design tokens.
- **Don't create custom modals or form controls.** Use shadcn/ui.
- **Don't skip the `"use client"` directive.** If the file uses hooks or event handlers, it needs it. If it doesn't, leave it as a server component.
- **Don't import Supabase client constructors directly.** Use the wrappers in `lib/supabase/`.
- **Don't persist genre strings without resolving** them through **`genresService.resolveLabelsToIds`** (or equivalent) into **`genre_ids`**.
- **Don't use `WidthType.PERCENTAGE`** — this is a docx concern, ignore it.
- **Don't assume Next.js APIs match your training data.** Read `node_modules/next/dist/docs/` first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pskoczenski)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pskoczenski)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
