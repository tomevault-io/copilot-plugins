## tadabro

> A web application for managing Quran recitation (Tadarus) groups. Members are assigned daily page ranges, mark tasks complete with proof images, and track collective progress.

# Tada-Bro — Project Reference

A web application for managing Quran recitation (Tadarus) groups. Members are assigned daily page ranges, mark tasks complete with proof images, and track collective progress.

---

## Tech Stack

| Layer                  | Technology                                   |
| ---------------------- | -------------------------------------------- |
| Framework              | Next.js 16 (App Router)                      |
| Language               | TypeScript 5                                 |
| Styling                | Tailwind CSS v4 + shadcn/ui (new-york style) |
| Database               | PostgreSQL via Drizzle ORM                   |
| Auth                   | JWT (jose) stored in httpOnly cookie         |
| File Storage           | Cloudflare R2 (S3-compatible, AWS SDK v3)    |
| Password hashing       | bcryptjs                                     |
| Runtime env validation | @t3-oss/env-nextjs + Zod                     |
| Icons                  | @phosphor-icons/react                        |

---

## Project Structure

```
app/
  (app)/                    # Authenticated routes (layout with sidebar)
    dashboard/              # Today's tasks + overdue items
    group/
      page.tsx              # Group list
      new/                  # Create group (super_admin only)
      [groupId]/
        page.tsx            # Group detail + daily tasks
        edit/               # Edit group name, description, settings
        progress/           # Group progress table (all members)
        task/[taskId]/      # Complete a task (upload proof image)
    profile/                # View/edit profile, change password
    admin/                  # Admin stats panel (super_admin only)
    admin/users/            # All users list + admin actions (super_admin only)
  api/
    auth/login, logout, register, me
    dashboard/              # Dashboard data (groups + today tasks + overdue tasks)
    admin/stats/            # Admin stats panel data (super_admin only)
    groups/[groupId]/       # CRUD + members + tasks + progress
    groups/[groupId]/detail # Rich group detail (group + members + today tasks)
    tasks/[taskId]/complete
    users/, users/[userId]
    profile/, profile/password
    avatar/, upload/
    invite/[token]/
  globals.css               # Tailwind + shadcn CSS variables
  layout.tsx
lib/
  db/
    index.ts                # Drizzle + pg Pool singleton
    schema.ts               # All table definitions + relations
    migrations/             # Drizzle SQL migrations
  utils/
    date.ts                 # todayLocal, formatDate, isToday, isPast, addDays, daysBetweenInclusive
    cn.ts                   # clsx + tailwind-merge helper
    invite.ts               # Invite token generation/hashing
components/
  auth/                     # LoginForm, RegisterForm
  group/                    # GroupCard, ProgressTable, TaskCard, etc.
  layout/                   # Sidebar, BottomNav
  ui/                       # Shared UI: Input, Button, Card, Badge, Textarea, Table, DropdownMenu, ...
scripts/
  seed-super-admin.ts       # Seeds initial super_admin user
  check-r2.ts               # Verifies R2 credentials + bucket reachability
  regenerate-group-tasks.ts # Regenerates daily tasks for a group
types/
  session.ts                # SessionPayload, SessionUser interfaces
env.ts                      # Validated env vars via @t3-oss/env-nextjs
```

---

## Database Schema

### Enums

- `user_role`: `super_admin | admin | member`
- `objective_type`: `whole_quran | page_range | surah_range | juz_range`
- `task_status`: `pending | complete | missed`

### Tables

**users**
| Column | Type | Notes |
|---|---|---|
| id | serial PK | |
| username | varchar(50) unique | |
| full_name | varchar(150) | |
| email | text unique | nullable (legacy accounts may not have it) |
| ic_number_encrypted | text unique | nullable; AES-256-GCM encrypted; format `hex(iv):hex(authTag):hex(ciphertext)` |
| ic_number_hash | varchar(64) unique | nullable; HMAC-SHA256 for uniqueness checks without decryption |
| password_hash | text | bcrypt |
| role | user_role | default `member` |
| avatar_url | text | nullable; R2 presigned URL key |
| created_at / updated_at | timestamptz | |

**groups**
| Column | Type | Notes |
|---|---|---|
| id | serial PK | |
| name | varchar(100) | |
| description | text | nullable |
| admin_id | int FK → users | |
| member_count | int | enforced ≤ 10 in API |
| objective_type | objective_type enum | |
| objective_meta | jsonb | null for whole_quran; `{startPage,endPage}` for page_range; `{startSurah,endSurah,startPage,endPage}` for surah_range; `{startJuz,endJuz,startPage,endPage}` for juz_range |
| total_pages | int | pre-computed at creation |
| start_date / end_date | date | YYYY-MM-DD |
| is_active | boolean | default true |
| allow_member_view | boolean | default false; allows members to see each other's proof images |
| created_at / updated_at | timestamptz | |

**group_members**

- `(group_id, user_id)` unique index
- `member_index` (0-based) determines page assignment order

**daily_tasks**

- `(group_id, user_id, task_date)` unique index
- `start_page`, `end_page`, `page_count` — computed at group creation / task regeneration
- `status`: `pending | complete | missed`

**task_completions**

- One-to-one with `daily_tasks` (taskId unique)
- Stores `proof_image_url` (R2 key) and `completed_at`

**invite_tokens**

- `token_hash` (SHA-256 of raw token; raw token never stored)
- `expires_at` (nullable = no expiry)
- `max_uses` (nullable = unlimited; typically set to `memberCount - 1`)
- `used_count` tracks consumption

---

## Roles & Permissions

| Action                                         | member | admin         | super_admin |
| ---------------------------------------------- | ------ | ------------- | ----------- |
| View own tasks/progress                        | ✓      | ✓             | ✓           |
| Complete tasks                                 | ✓      | ✓             | ✓           |
| View other members' proof (if allowMemberView) | ✓      | ✓             | ✓           |
| Create/edit group                              | —      | ✓ (own group) | ✓           |
| Manage group members                           | —      | ✓ (own group) | ✓           |
| Admin panel                                    | —      | —             | ✓           |
| All users list                                 | —      | —             | ✓           |
| Reset any user's password                      | —      | —             | ✓           |

---

## Authentication Flow

1. Login → `POST /api/auth/login` → JWT signed with `AUTH_SECRET`, set as `session` httpOnly cookie (7 day expiry)
2. Protected routes call `getSession()` which verifies the JWT
3. `GET /api/auth/me` returns full `SessionUser` (id, username, fullName, email, role, avatarUrl)
4. Logout → `POST /api/auth/logout` → clears cookie
5. Registration via invite token: `GET /api/invite/[token]` validates token → `POST /api/auth/register` creates user and joins group

**SessionUser type:**

```ts
interface SessionUser {
  id: number
  username: string
  fullName: string
  email?: string | null
  role: 'super_admin' | 'admin' | 'member'
  avatarUrl: string | null
}
```

---

## API Routes

| Method           | Path                             | Description                                                                                             |
| ---------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------- |
| POST             | `/api/auth/login`                | Login, set JWT cookie                                                                                   |
| POST             | `/api/auth/logout`               | Clear session cookie                                                                                    |
| POST             | `/api/auth/register`             | Register via invite token                                                                               |
| GET              | `/api/auth/me`                   | Get current session user                                                                                |
| GET              | `/api/invite/[token]`            | Validate invite token + get group info                                                                  |
| GET              | `/api/dashboard`                 | Groups + today tasks + overdue tasks; `?today=` (client local date); returns `currentUserId`, `isAdmin` |
| GET              | `/api/admin/stats`               | All groups + user count + completed count (super_admin)                                                 |
| GET/POST         | `/api/groups`                    | List / create groups                                                                                    |
| GET/PATCH/DELETE | `/api/groups/[groupId]`          | Get / update / delete group                                                                             |
| GET              | `/api/groups/[groupId]/detail`   | Rich group detail: members + today tasks; `?today=`; returns `currentUserId`, `isAdmin`                 |
| GET/POST         | `/api/groups/[groupId]/members`  | List / add members                                                                                      |
| GET              | `/api/groups/[groupId]/tasks`    | Get daily tasks for group                                                                               |
| GET              | `/api/groups/[groupId]/progress` | Progress table data; `?today=`, `?date=`, `?userId=`, `?page=`; paginated (10/page)                     |
| GET              | `/api/tasks/[taskId]`            | Get single task                                                                                         |
| POST             | `/api/tasks/[taskId]/complete`   | Submit proof image, mark complete                                                                       |
| GET/PATCH        | `/api/profile`                   | Get / update profile                                                                                    |
| POST             | `/api/profile/password`          | Change own password                                                                                     |
| GET              | `/api/users`                     | All users (super_admin)                                                                                 |
| GET/PATCH/DELETE | `/api/users/[userId]`            | User detail / update / delete (super_admin)                                                             |
| POST             | `/api/avatar`                    | Upload avatar to R2                                                                                     |
| POST             | `/api/upload`                    | General file upload to R2                                                                               |

---

## File Storage (Cloudflare R2)

- SDK: `@aws-sdk/client-s3` + `@aws-sdk/s3-request-presigner`
- Endpoint: `https://${R2_ACCOUNT_ID}.r2.cloudflarestorage.com`
- Region: `"auto"`
- Used for: proof images (task completions), avatars
- Check connection: `npm run r2:check:dev` / `npm run r2:check:prod`

---

## Styling Architecture

### Tailwind CSS v4 + shadcn/ui

The project uses two parallel theme systems that must stay in sync:

**Project custom tokens** — defined in `app/globals.css` `@theme {}` block:

```css
--color-primary:
  #edac3b /* golden yellow */ --color-primary-dark: #b47d1e
    --color-primary-light: #f5c842 --color-bg,
  --color-surface, --color-surface-raised --color-fg, --color-muted-fg,
  --color-border;
```

**shadcn CSS variables** — defined in `:root` / `.dark` blocks; must be kept consistent with project tokens:

```css
--primary: #edac3b /* must match --color-primary */ --background: var(--bg)
  --foreground: var(--fg) --popover: var(--surface)
  --accent: var(--surface-raised) --muted-foreground: var(--muted-fg)
  --input: var(--surface) /* white input background in light mode */;
```

> **Critical**: `@theme inline {}` (added by shadcn init) takes precedence over `@theme {}` for shared keys. Always define shadcn variables in `:root`/`.dark` pointing to project tokens, not as hardcoded oklch values.

### Component conventions

- `text-muted-foreground` — for subdued/inactive text (maps to `#78716c`)
- `text-muted` — do NOT use for text; this is a background-level muted color (`oklch(0.97 0 0)` = near-white, invisible on white backgrounds)
- `bg-surface` / `bg-surface-raised` — card and hover backgrounds
- `text-primary-dark` / `dark:text-primary-light` — active link/selected state text

---

## Scripts

| Script           | Command                                           | Description                                                  |
| ---------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| Dev server       | `npm run dev`                                     | Next.js dev server                                           |
| DB generate      | `npm run db:generate:dev`                         | Drizzle generate migration (dev)                             |
| DB migrate       | `npm run db:migrate:dev`                          | Apply migrations (dev)                                       |
| DB studio        | `npm run db:studio:dev`                           | Drizzle Studio UI (dev)                                      |
| Seed super admin | `npm run seed:dev`                                | Create initial super_admin user                              |
| Check R2         | `npm run r2:check:dev`                            | Verify R2 credentials + bucket access                        |
| Regen tasks      | `npm run tasks:regenerate:dev`                    | Regenerate daily tasks for a group                           |
| Check DB         | `npm run db:check:dev`                            | Verify DB connection + schema tables                         |
| Backup DB        | `npm run db:backup:dev`                           | pg_dump to `backups/backup-*.sql`                            |
| Restore DB       | `npm run db:restore:prod -- backups/backup-*.sql` | psql restore to `DATABASE_URL_NEW`                           |
| Migrate R2 paths | `npm run db:migrate-r2-paths:dev`                 | Strip R2_PUBLIC_URL prefix from proof_image_url + avatar_url |

All scripts have `:dev` and `:prod` variants using `.env.development` or `.env.production` respectively.

---

## Environment Variables

Required in `.env.development` / `.env.production`:

```env
DATABASE_URL=
JWT_SECRET=
NODE_ENV=

# App URL (used for server-side API fetches in page components)
NEXT_PUBLIC_APP_URL=

# Cloudflare R2
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=
R2_PUBLIC_URL=

# IC encryption (legacy; columns now nullable)
ENCRYPTION_KEY=
HMAC_SECRET_KEY=

# Target database for migration (used by db:restore:prod only)
DATABASE_URL_NEW=
```

---

## Key Utilities

**`lib/utils/date.ts`**

- `todayLocal()` — today as `YYYY-MM-DD` in the client's runtime timezone (`new Date().toLocaleDateString("en-CA")`)
- `isToday(dateStr)` / `isPast(dateStr)` — compare against `todayLocal()`
- `formatDate(dateStr)` — `"15 Jan 2024"` format
- `daysBetweenInclusive(start, end)` — inclusive day count
- `addDays(dateStr, n)` — add n days, returns `YYYY-MM-DD`

**`lib/utils/cn.ts`** — `cn(...classes)` combining clsx + tailwind-merge (also exported from `lib/utils.ts` for shadcn components)

**`lib/utils/invite.ts`** — invite token generation and SHA-256 hashing

---

## Page Data-Fetching Convention

All pages fetch data exclusively through API routes — no direct DB imports in pages.

### Server component pages (e.g. `/group/new`, `/admin`)

Call `getSession()` for early redirect guard, then forward the session cookie to the API:

```ts
import { cookies } from 'next/headers'
import { env } from '@/env'

const session = await getSession()
if (!session) redirect('/login')

const cookieStore = await cookies()
const res = await fetch(`${env.NEXT_PUBLIC_APP_URL}/api/…`, {
  headers: { cookie: cookieStore.toString() },
  cache: 'no-store',
})
```

### Client component pages (dashboard, group detail, progress)

Pages that need the client's local date (`todayLocal()`) for timezone-correct task queries are `'use client'` components. They fetch data in `useEffect` with relative URLs — cookies are sent automatically by the browser (same-origin):

```ts
'use client'
import { useEffect, useState } from 'react'
import { todayLocal } from '@/lib/utils/date'

useEffect(() => {
  fetch(`/api/dashboard?today=${todayLocal()}`)
    .then((res) => res.json())
    .then((json) => setData(json.data))
}, [])
```

Auth guard is handled by `app/(app)/layout.tsx` — client pages do not call `getSession()`. User identity (`currentUserId`, `isAdmin`) is returned by the API response instead.

APIs that accept `?today=` use it for date-sensitive queries and fall back to server-side `todayLocal()` if the param is absent.

---

## Page Reference

| Route                       | Access                              | Description                                                         |
| --------------------------- | ----------------------------------- | ------------------------------------------------------------------- |
| `/dashboard`                | all                                 | Today's tasks; overdue tasks; group summary                         |
| `/group`                    | all                                 | List of joined groups                                               |
| `/group/new`                | admin / super_admin                 | Create a new Tadarus group                                          |
| `/group/[id]`               | member                              | Group detail; daily task list                                       |
| `/group/[id]/edit`          | admin/super_admin                   | Edit name, description, allowMemberView                             |
| `/group/[id]/progress`      | member (if allowMemberView) / admin | Progress table with server-side member + date filter and pagination |
| `/group/[id]/task/[taskId]` | member                              | Upload proof image to complete task                                 |
| `/profile`                  | all                                 | View/edit profile; change password; view email (read-only)          |
| `/admin`                    | super_admin                         | Admin stats panel                                                   |
| `/admin/users`              | super_admin                         | All users; reset passwords                                          |

---
> Source: [HaziziHamdan/tadabro](https://github.com/HaziziHamdan/tadabro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
