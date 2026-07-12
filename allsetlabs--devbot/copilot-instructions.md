## devbot

> Personal assistant mobile app that acts as a Claude Code terminal proxy — trigger development tasks from your phone anywhere.

# DevBot

## Goal

Personal assistant mobile app that acts as a Claude Code terminal proxy — trigger development tasks from your phone anywhere.

## Description

Users open the DevBot mobile app to launch a web-based terminal (xterm.js) connected to a Node.js backend that manages Claude Code sessions via tmux. The backend supports a plugin system where each plugin (baby-logs, lawn-care, etc.) is self-contained with its own SQLite database, routes, and schema. Sessions are persisted via Supabase.

## Architecture

```
devbot/
├── app/         # Web app (Vite + React + xterm.js terminal UI)
├── backend/     # Node.js session manager (xterm-ws + tmux + Claude Code runner)
├── supabase/    # Local Supabase (session persistence + migrations)
├── intro-video/ # Remotion intro video project
└── plugins/     # Plugin modules (baby-logs, lawn-care, etc.)
```

Shared component library: `@allsetlabs/forge` (`forge-modules/forge`, sibling module).

Stack: Vite + React + TypeScript (app), Node.js + TypeScript + Express (backend), Drizzle ORM + SQLite (per-plugin DBs), Supabase (session storage).

## Progress

Core terminal proxy is functional. Plugin system is live with baby-logs and lawn-care. Scheduler system runs automated Claude tasks. Interactive chats with message queuing are implemented.

## Autonomy

DevBot sessions are automated — the user is often away. **Minimize questions.** Make a decision and run. Only ask when a wrong guess would cause real damage (deleting data, wrong module). For ambiguous choices, pick the best option and move forward.

## Critical: Do Not Restart DevBot

**Never restart DevBot without explicit user permission.** No `kill`, no `tmux send-keys C-c`, no manual `npm run dev`. Active Claude workers and chats will be dropped. The only safe restart is `make start` from the repo root, run in an external Terminal.

## Database Architecture

Uses **Drizzle ORM + SQLite** with a database-per-plugin pattern:

- **`devbot.db`** — Core features (sessions, chats, schedulers, plans)
- **`baby-logs.db`** — Baby Logs plugin
- **`lawn-care.db`** — Lawn Care plugin

Each plugin owns its SQLite file and self-initializes via `db.ts`.

### Standard Columns (All Tables)

Every table MUST include:

```ts
created_by: text('created_by').notNull().default('user');
created_at: text('created_at')
  .notNull()
  .$defaultFn(() => new Date().toISOString());
updated_by: text('updated_by').notNull().default('user');
updated_at: text('updated_at')
  .notNull()
  .$defaultFn(() => new Date().toISOString());
settings: text('settings', { mode: 'json' }).$type<Record<string, any>>().default({});
```

### JSONB `settings` Column Strategy

Push optional/flexible fields into `settings` instead of adding new columns — avoids schema migrations and backend restarts:

```ts
// Good: no migration needed
settings: { model: 'sonnet', temperature: 0.7 }
// Access: chat.settings.model ?? 'sonnet'
```

## Plugin Architecture

**Location:** `plugins/{plugin-name}/`

**Table naming:** All plugin tables MUST be prefixed with `{plugin_name}_` (e.g., `baby_logs_logs`, `lawn_care_plans`).

**Structure:**

```
plugins/{plugin-name}/
├── src/
│   ├── backend/
│   │   ├── schema.ts    # Drizzle schema with prefixed table names
│   │   ├── handlers/    # API handlers split by domain
│   │   ├── routes.ts    # export default function getRouter(db): Router
│   │   └── types.ts     # TypeScript types from schema
│   └── index.ts         # Barrel export
└── package.json
```

**Router factory pattern:**

```ts
export default function getBabyLogsRouter(db: DrizzleInstance): Router {
  const router = Router();
  router.get('/logs', async (req, res) => { ... });
  return router;
}
```

Plugin registration in backend `index.ts`:

```ts
app.use('/api/plugins/baby-logs', getBabyLogsRouter());
```

### Mutation Best Practices

Always set `created_by`/`updated_by` to identify the source on insert/update:

```ts
await db.insert(table).values({ ..., created_by: 'user', updated_by: 'user', settings: {} });
await db.update(table).set({ updated_by: 'AI' }).where(eq(table.id, id));
```

### Migration Notes

- Schema defined in TypeScript (Drizzle), not SQL
- Databases auto-initialize on startup via `initializeCoreDatabase()`
- No separate migration files during development
- Foreign keys enabled: `PRAGMA foreign_keys = ON`

## Frontend Coding Standards

### Mobile-First Hover Handling

**Problem:** Hover states don't work on mobile touch devices. Users expect click-based interactions instead.

**Pattern:** Disable hover effects on mobile screens, use responsive Tailwind breakpoints.

```tsx
// BAD - hover works everywhere, confusing on mobile where hover isn't possible
<button className="hover:bg-blue-500">Click me</button>

// GOOD - hover only on desktop (md breakpoint = 768px+), click works everywhere
<button className="md:hover:bg-blue-500 active:bg-blue-700">Click me</button>

// GOOD - interactive elements should always have click handlers, not hover-only
<span
  className="md:hover:opacity-80 transition-opacity cursor-pointer"
  onClick={handleClick}
  onKeyDown={handleKeyDown}
/>
```

**Guidelines:**

- Prefix hover classes with `md:` to disable on mobile (screens < 768px)
- Always provide a click handler (`onClick`) as the primary interaction
- Add visual feedback on both hover (desktop) and click (mobile): use `active:` or managed state
- Use `transition-opacity` or `transition-colors` for smooth feedback
- Avoid `hover:` without the `md:` prefix for interactive elements
- For touch-only interactions, use `focus:` instead of `hover:`

---
> Source: [allsetlabs/devbot](https://github.com/allsetlabs/devbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
