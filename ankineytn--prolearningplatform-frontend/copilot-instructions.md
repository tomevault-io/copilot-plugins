## prolearningplatform-frontend

> React 19 + TypeScript + Vite SPA for an online learning platform. See README.md for full feature list and tech stack.

# CLAUDE.md — ProLearning Frontend

## Project Overview

React 19 + TypeScript + Vite SPA for an online learning platform. See README.md for full feature list and tech stack.

## Dev Commands

```bash
npm run dev       # Start dev server on http://localhost:3000
npm run build     # tsc -b && vite build
npm run lint      # ESLint
npm run preview   # Preview production build
```

## Architecture

### State Management

**No Redux.** Server state is exclusively managed with **TanStack React Query v5**.

- Query cache is persisted to `localStorage` via `createAsyncStoragePersister` with 24h TTL
- Auth and notifications are excluded from persistence (volatile data)
- Query keys follow a per-feature pattern: `['sets']`, `['flashcards', setId]`, etc.
- Mutations call `queryClient.invalidateQueries()` to trigger refetches

### API Layer

```
services/
├── client.ts          # Axios instance with JWT interceptors
├── endpoints/         # One module per feature (sets.ts, flashcard.ts, exam.ts …)
└── types/             # TypeScript interfaces for API responses
```

- `client.ts` exports two instances: `apiClient` (auth headers) and `publicApiClient` (no auth)
- Request interceptor: injects `Authorization: Bearer {accessToken}`
- Response interceptor: silently refreshes token on 401, redirects to `/login` if refresh fails
- Tokens stored in `localStorage`

### Hooks

All data-fetching logic lives in `src/hooks/`. Each hook:

1. Calls functions from `services/endpoints/`
2. Wraps them in `useQuery` / `useMutation`
3. Exposes typed data + loading/error states to components

Never call API functions directly in components — always go through hooks.

### Routing

Defined in `src/config/routeConfig.tsx`. Three route groups:

- **Public routes** — auto-redirect to dashboard if authenticated
- **Protected routes** — wrapped in `ProtectedLayout` which checks `useAuth()`
- **Admin routes** — additionally check `ROLE_ADMIN`

### Real-time Collaboration

Notes use Yjs CRDTs synced over WebSocket:

- Provider: Hocuspocus (`VITE_COLLAB_WS_URL`)
- Hook: `useCollaboration`
- Only active on the Note editor page

### Styling

- Tailwind CSS v4 (configured via `@tailwindcss/vite`)
- Radix UI headless primitives + custom styles in `src/components/ui/`
- Dark/light theme via `ThemeProvider` — persisted to `localStorage`
- Global CSS in `src/index.css`

### Forms

React Hook Form + Zod. Schemas live in `src/schemas/`. Always use the `@hookform/resolvers/zod` resolver.

### i18n

Two languages: `en` and `vi`. Translation files at `src/i18n/resources/`. Use the `useTranslation` hook; never hardcode user-facing strings.

## Key Conventions

- **Path alias**: `@/` maps to `src/` — use it for all imports
- **Component files**: PascalCase `.tsx`; hooks: camelCase prefixed with `use`
- **No comments** unless the WHY is non-obvious
- **Feature docs**: `FLASHCARD_GUIDE.md`, `REVIEW_SYSTEM.md`, `KNOWLEDGE_ANALYSIS_FE_INTEGRATION.md`, `ROADMAP_FE_INTEGRATION.md` have detailed API integration notes for those subsystems

## Environment Variables

```
VITE_API_URL          # Backend REST API base URL
VITE_COLLAB_WS_URL    # WebSocket server for collaborative editing
```

All env vars must be prefixed with `VITE_` to be accessible via `import.meta.env`.

---

## Design System

### Color Themes

Three themes applied via `data-color-theme` on `<html>`: `atlas` (green), `lumen` (blue), `ember` (orange). All tokens are defined in `src/index.css` under the `--pl-*` namespace with light/dark variants.

**Never hardcode hex, rgb, or oklch values in components. Always use CSS variables.**

#### Semantic tokens

| Token                 | Purpose                                                |
| --------------------- | ------------------------------------------------------ |
| `--pl-accent`         | Theme-adaptive accent — changes with atlas/lumen/ember |
| `--pl-accent-soft`    | Accent ~12% opacity — subtle backgrounds               |
| `--pl-accent-soft-2`  | Accent ~7% opacity — card/row backgrounds              |
| `--pl-accent-border`  | Accent ~32% opacity — borders                          |
| `--pl-accent-strong`  | Darker accent — bold/emphasized text                   |
| `--pl-accent-fg`      | Foreground colour on a solid accent background         |
| `--pl-danger`         | Error/destructive — always red, not theme-dependent    |
| `--pl-danger-soft`    | Red soft bg (dark-aware)                               |
| `--pl-danger-border`  | Red border ~35% opacity                                |
| `--pl-danger-text`    | Red text (dark-aware)                                  |
| `--pl-warning`        | Caution/pending — always amber                         |
| `--pl-warning-soft`   | Amber soft bg (dark-aware)                             |
| `--pl-warning-border` | Amber border ~40% opacity                              |
| `--pl-warning-text`   | Amber text (dark-aware)                                |
| `--pl-success`        | Fixed success green (not theme-dependent)              |
| `--pl-success-soft`   | Success soft bg                                        |
| `--pl-bg`             | Page background                                        |
| `--pl-bg-elev`        | Elevated surface (cards, panels)                       |
| `--pl-bg-hover`       | Hover / sunken surface                                 |
| `--pl-bg-sunken`      | Deeply sunken (editor areas)                           |
| `--pl-border`         | Default border                                         |
| `--pl-border-strong`  | Prominent border                                       |
| `--pl-text`           | Primary text                                           |
| `--pl-text-muted`     | Secondary text                                         |
| `--pl-text-faint`     | Tertiary / label text                                  |

#### Status colour convention

Apply this consistently across **every** feature — forms, tables, cards, badges, icons:

| State             | Tokens           | Example use                             |
| ----------------- | ---------------- | --------------------------------------- |
| Correct / success | `--pl-accent-*`  | correct answer, completed step          |
| Error / wrong     | `--pl-danger-*`  | validation error, wrong answer          |
| Pending / warning | `--pl-warning-*` | ungraded, in-progress                   |
| Neutral success   | `--pl-success-*` | general "done" that isn't theme-branded |

### Tailwind vs Inline Styles

Tailwind v4 supports arbitrary CSS-variable values. **Always prefer `className`** over `style={{}}`.

```tsx
// ✅ correct
<div className='bg-[var(--pl-accent-soft)] border-[var(--pl-accent-border)] text-[var(--pl-accent-strong)]' />

// ❌ avoid
<div style={{ background: 'var(--pl-accent-soft)', borderColor: 'var(--pl-accent-border)' }} />
```

`style={{}}` is acceptable **only when Tailwind cannot express it**:

- Complex gradients: `radial-gradient(...)`, `linear-gradient(...)`
- SVG `filter`: `drop-shadow(0 0 8px var(--pl-accent-soft))`
- Dynamic numeric values computed at runtime: `strokeDasharray`, `width: collapsed ? 68 : 232`
- `onMouseEnter` / `onMouseLeave` JS handlers → replace with `hover:` Tailwind classes instead

For repeated conditional class strings, use an `as const` map rather than inline ternaries:

```tsx
const STATUS_CLASS = {
  correct:
    'border-[var(--pl-accent-border)] bg-[var(--pl-accent-soft-2)] text-[var(--pl-accent-strong)]',
  wrong:
    'border-[var(--pl-danger-border)] bg-[var(--pl-danger-soft)] text-[var(--pl-danger-text)]',
  pending:
    'border-[var(--pl-warning-border)] bg-[var(--pl-warning-soft)] text-[var(--pl-warning-text)]',
} as const;
```

### Typography

```tsx
// Hero titles, large numbers, italic headings
className = 'font-[family-name:var(--font-display)] italic';

// Section headers
className = 'font-[family-name:var(--font-display)] text-[26px] font-normal';

// Mono labels — always UPPERCASE with letter-spacing
className =
  'font-[family-name:var(--font-mono-pl)] text-[11px] tracking-[0.2em] text-[var(--pl-text-faint)]';
```

### Cards & Containers

**Primary content card** — for any card-level container:

```tsx
className =
  'rounded-2xl border border-[var(--pl-border)] bg-[var(--pl-bg)] p-5';
```

**Elevated card** — panels, tables, sidebar panels:

```tsx
className =
  'rounded-2xl border border-[var(--pl-border)] bg-[var(--pl-bg-elev)]';
```

**Smaller UI components** (tags, list items, inline chips):

```tsx
className = 'rounded-[14px] border border-[var(--pl-border)]';
```

Rules:

- Never use `rounded-lg border-2` for cards.
- `rounded-2xl` = primary content. `rounded-[14px]` = secondary / inline. `rounded-full` = buttons, avatars, badges only.
- Use `border-[var(--pl-border)]` — never `border-border` — for explicit theme control.

**Icon container** inside cards:

```tsx
className =
  'w-9 h-9 rounded-lg flex items-center justify-center bg-[var(--pl-accent-soft)]';
```

**Badge / status pill**:

```tsx
className='inline-flex items-center text-[11px] px-2 py-0.5 rounded-full border
           border-[var(--pl-accent-border)] bg-[var(--pl-accent-soft)] text-[var(--pl-accent)]'
```

### Hero Section Pattern

Used on result pages and any full-width summary card:

```tsx
<div
  className='relative rounded-2xl border border-[var(--pl-border)] bg-[var(--pl-bg)] overflow-hidden p-10 text-center'
  style={{
    background:
      'radial-gradient(circle at 50% 40%, var(--pl-accent-soft), transparent 60%)',
  }}
>
  <h2 className='font-[family-name:var(--font-display)] italic text-5xl font-normal tracking-tight mb-3'>
    {title}
  </h2>
  <p className='text-muted-foreground text-sm mb-8 max-w-md mx-auto'>
    {subtitle}
  </p>
  {/* SVG ring / visual / metric */}
  {/* CTA buttons, centered, flex-wrap */}
</div>
```

### Stat Cards (3-column grid)

Standard layout for any metrics summary:

```tsx
<div className='grid grid-cols-1 md:grid-cols-3 gap-4'>
  <div className='rounded-2xl border border-[var(--pl-border)] bg-[var(--pl-bg)] p-5'>
    {/* top row: icon square + badge pill */}
    <div className='flex items-center justify-between mb-4'>
      <div className='w-9 h-9 rounded-lg flex items-center justify-center bg-[var(--pl-accent-soft)]'>
        <Icon className='w-4 h-4 text-[var(--pl-accent)]' />
      </div>
      <span className='... badge pill classes ...'>label</span>
    </div>
    {/* mono label */}
    <p className='font-[family-name:var(--font-mono-pl)] text-[11px] tracking-[0.2em] text-muted-foreground mb-1'>
      METRIC NAME
    </p>
    {/* display number */}
    <p className='font-[family-name:var(--font-display)] text-3xl font-medium mb-1'>
      42
    </p>
    {/* sub-label */}
    <p className='text-xs text-muted-foreground'>supporting text</p>
  </div>
</div>
```

### Page Layout

**Standard scrollable page:**

```tsx
<div className='min-h-screen bg-[var(--pl-bg)]'>
  {/* sticky header */}
  <main className='max-w-5xl mx-auto px-6 py-10 space-y-6'>
    {/* content sections */}
  </main>
</div>
```

**Wide dashboard / list page:** `max-w-[1400px]`
**Narrow focused page (results, forms):** `max-w-4xl`
**Fixed-height editor/split layouts:** Only for pages that require it (NotePage). Use `h-[calc(100dvh-Xpx)] flex flex-col overflow-hidden`.

### Page Headers

```tsx
{
  /* breadcrumb label */
}
<p className='font-[family-name:var(--font-mono-pl)] text-[11px] tracking-[0.18em] text-[var(--pl-text-faint)] uppercase mb-1'>
  Section name
</p>;
{
  /* page title */
}
<h1 className='font-[family-name:var(--font-display)] text-[32px] font-normal mb-2'>
  Page Title
</h1>;
{
  /* subtitle */
}
<p className='text-[14px] text-[var(--pl-text-muted)]'>Subtitle</p>;
```

### Empty States

```tsx
<div className='flex flex-col items-center justify-center py-24 text-center'>
  <div className='w-20 h-20 rounded-[20px] grid place-items-center mb-5 bg-[var(--pl-accent-soft)]'>
    <Icon size={36} className='text-[var(--pl-accent)]' />
  </div>
  <h3 className='text-[20px] font-medium mb-2'>{title}</h3>
  <p className='text-[13px] text-[var(--pl-text-muted)] mb-6 max-w-[440px]'>
    {description}
  </p>
  <Button>{cta}</Button>
</div>
```

### Loading States

```tsx
{
  /* full-page */
}
<div className='min-h-screen flex items-center justify-center bg-[var(--pl-bg)]'>
  <div className='w-8 h-8 border-2 border-[var(--pl-accent)] border-t-transparent rounded-full animate-spin' />
</div>;

{
  /* inline / section */
}
<div className='flex items-center gap-2 text-[var(--pl-text-muted)]'>
  <Loader2 className='w-4 h-4 animate-spin' />
  <span className='text-sm'>{loadingText}</span>
</div>;
```

---
> Source: [AnkineyTN/ProlearningPlatform_Frontend](https://github.com/AnkineyTN/ProlearningPlatform_Frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
