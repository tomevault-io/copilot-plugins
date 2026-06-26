## to-do

> A task management app with multiple workspaces, calendar view, and Supabase persistence.

# TO-DO

A task management app with multiple workspaces, calendar view, and Supabase persistence.

## Tech Stack

- **React 19** + **TypeScript 5.9**
- **Vite 8** — build tool
- **HeroUI 3** — component library (use `@heroui/react` imports)
- **Tailwind CSS 4** — styling (no config file, configured via `src/index.css`)
- **Supabase** — backend (tasks + workspaces)
- **@dnd-kit/core + @dnd-kit/sortable** — drag and drop
- **date-fns + rrule** — date handling and recurrence

## Project Structure

```
src/
  assets/icons.tsx          # All SVG icons (DragHandle, Trash, Pencil, Hamburger, Chevron)
  data/
    taskRepository.ts               # LocalStorage task repo (legacy/fallback)
    workspaceRepository.ts          # LocalStorage workspace helpers + migration logic
    supabaseTaskRepository.ts       # Supabase task repo (active)
    supabaseWorkspaceRepository.ts  # Supabase workspace repo (active)
  features/todos/
    TodoApp.tsx       # Root shell — sidebar drawer, tab switching, workspace state
    TodoList.tsx      # List view — drag-to-reorder, inline edit, completed section
    CalendarMonth.tsx # Calendar view — drag-to-reschedule
    Sidebar.tsx       # Workspace list drawer
    TaskProvider.tsx  # Context provider — wraps repo calls, triggers refresh
    tasksContext.ts   # React context type definition
    useTasks.ts       # Hook to consume TasksContext
  lib/
    supabase.ts       # Supabase client (reads from env)
    recurrence.ts     # rrule helpers for recurring tasks
  types/task.ts       # Task, CreateTaskInput, UpdateTaskInput types
```

## Environment Variables

Copy `.env.example` to `.env` and fill in values:

```
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
```

## Supabase Schema

```sql
create table workspaces (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  created_at timestamptz default now()
);

create table tasks (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid references workspaces(id) on delete cascade,
  title text not null,
  completed boolean default false,
  due_date date,
  recurrence jsonb,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);
```

## Key Patterns

- **Repository pattern** — `TaskRepository` interface in `taskRepository.ts`. Swap implementations (localStorage ↔ Supabase) without touching UI.
- **TaskProvider keyed by workspaceId** — switching workspaces fully remounts the provider, preventing state bleed.
- **Task order** — stored separately in `localStorage` under `hero-todos:order:{workspaceId}`. Supabase does not store order.
- **Dark mode** — HeroUI's `dark:` variant responds to `prefers-color-scheme`. Avoid `dark:` classes on calendar cells to prevent OS dark mode conflicts.

## Commands

```bash
npm run dev      # Start dev server
npm run build    # Production build
npm run lint     # ESLint
```

---
> Source: [kannanvg100/to-do](https://github.com/kannanvg100/to-do) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
