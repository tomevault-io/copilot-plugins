## kosmos0

> > **This file governs V88 solutions built with Daia (Deployed AI Architecture)** — V88's solution template, distributed via the `v88-ai-solution-template` GitHub repo. Daia ships a runnable starter app — auth, user management, avatar menu, notifications, glass design system, PWA manifest. Your behaviour depends on whether you're **maintaining Daia itself** or **building an app on top of it** — Step 0 below tells you which.

# CLAUDE.md — V88 Project

> **This file governs V88 solutions built with Daia (Deployed AI Architecture)** — V88's solution template, distributed via the `v88-ai-solution-template` GitHub repo. Daia ships a runnable starter app — auth, user management, avatar menu, notifications, glass design system, PWA manifest. Your behaviour depends on whether you're **maintaining Daia itself** or **building an app on top of it** — Step 0 below tells you which.
>
> In APP-BUILD mode, your job is to:
>
> 1. Detect the current state of the project (state detection below)
> 2. Either run starter setup, wait for scope, or build the app — exactly one of those
> 3. Never duplicate what the starter already gives you

---

## Pre-Flight Checks

> **STOP. Before generating any code, run Step 0 to determine which mode you're in. Everything that follows depends on it.**

### Step 0 — detect mode (every session, before anything else)

Run **both** of these. Use the strongest signal — either one matching is enough.

```bash
basename "$(pwd)"
basename -s .git "$(git remote get-url origin 2>/dev/null)"
```

| Result | Mode |
|---|---|
| Either output is exactly `v88-ai-solution-template` | **TEMPLATE-MAINTENANCE MODE** |
| Anything else | **APP-BUILD MODE** |

#### If TEMPLATE-MAINTENANCE MODE

- **Do NOT continue with the state-detection checks below.** They are app-build only.
- **Read `TEMPLATE.md` IMMEDIATELY — before any other action.** It contains the **Template Change Protocol** that governs every modification to this repo. Do not write any code, edit any file, or propose any change until you have classified the request per the protocol's four-category framework and confirmed scope with the user.
- Every change to `src/`, `supabase/`, `skills/`, `public/`, `scripts/`, `CLAUDE.md`, `TEMPLATE.md`, or `README.md` follows the protocol — there are no exceptions for "small" changes (small changes are exactly where drift creeps in).
- The skill-editing prohibition in the Boundaries section below does **not** apply to you — you ARE the maintainer.

#### If APP-BUILD MODE

- Continue to "State detection" below.
- The Boundaries section applies fully — never edit `SKILL.md` files, never make ad-hoc schema changes, etc.

---

### State detection (APP-BUILD mode only)

Use three signals to place the project on the matrix:

**1. Supabase schema state** — via Supabase MCP, call `list_tables` on the `public` schema.
- No `profiles` table → **schema is empty** (starter not yet applied)
- `profiles` + `notifications` present → **starter applied**

**2. Scope state** — read `V88-SCOPE.md`.
- Any section still contains `[TODO]` → **scope not populated**
- All sections filled (app name, brand colour, entities, features) → **scope populated**

**3. App icon state** — check whether `v88_context/app-icon.png` exists.
- File present → **icon ready**
- File missing → **icon missing** (still needed before rebrand can run)

### Four states, four paths

| Schema | Scope | Icon | What to do |
|--------|-------|------|------------|
| Empty | Any | Any | **Run starter setup.** Do NOT read `V88-SCOPE.md` yet. Invoke the `starter-setup` skill. |
| Applied | `[TODO]` present | Any | **Wait for scope.** Starter works — tell the user to log in, try it, then fill in `V88-SCOPE.md`. |
| Applied | Populated | Missing | **Wait for icon.** Tell the user: *"Drop a square PNG (≥ 128 px, recommended ≥ 512 px) at `v88_context/app-icon.png` so I can generate every platform variant during rebrand."* |
| Applied | Populated | Present | **Run app pre-flight**, confirm with user, rebrand the starter, then build. See "App pre-flight" below. |

### Starter setup (schema empty)

Invoke the `starter-setup` skill and follow it exactly. Summary of what it does:

1. Confirm the Supabase project URL with the user
2. `npm install`
3. Write `.env.local` with URL + anon key (retrieved via MCP)
4. Apply all three migrations in order
5. Deploy all four edge functions
6. Invoke `bootstrap-admin` to seed `admin@v88.co.uk / Admin!123` with welcome notification
7. Start `npm run dev` in background, tell the user to log in at `http://localhost:5173`

(See `skills/starter-setup/SKILL.md` for the canonical, numbered step list — including the VAPID + `app_secrets` seed needed before push works.)

After setup, tell the user:

> "Setup complete. Log in, try it, then fill in `V88-SCOPE.md` and come back to build your app."

Do not proceed to app code until scope is populated.

### Starter applied, scope has `[TODO]`

Don't build anything. Say something like:

> "The starter is already set up. Log in at http://localhost:5173 with `admin@v88.co.uk / Admin!123`, poke around (Users, Notifications, Profile), then fill in `V88-SCOPE.md` with your app name, brand colour, entities, and features. Come back when that's done."

### App pre-flight (starter applied, scope populated)

Now you're building the app on top of the starter. Before writing code:

1. **Read `V88-SCOPE.md`** carefully — app name, brand colour (HSL), entities, features. Also check `v88_context/` for logos, brand assets, transcriptions, or mockups. Factor them in.
2. **Present your understanding back to the user** — entity list, relationships, features, architectural decisions. Explicitly ask:
   - *"Based on the entity relationships, I'll subscribe [child tables] alongside [parent tables] on the relevant pages so hierarchy changes propagate instantly. Does this look right?"*
   - *"Are there any tables whose values affect calculations or behaviour across multiple screens — a config table with rates or rules? I'll wire those as app-level Realtime subscriptions in `AppLayout`'s `foundationalSubs` array."*
3. **Rebrand the starter** before app-specific entity work. Update these to match the project's brand and app name:
   - `src/globals.css` — `--primary`, `--accent`, `--ring`, `--sidebar-primary`, `--sidebar-ring` HSL values + `.btn-brand` gradient hex values
   - `src/components/AppSidebar.tsx` — the `"V8"` logo text, `"V88 App"` title, and the `rgba(63,186,194,…)` active-state tint (match your new brand)
   - `src/components/AppLayout.tsx` — the mobile top bar's `"V8"` + `"V88 App"`
   - `src/components/MobileTabBar.tsx` — `ACTIVE_COLOR`
   - `src/pages/LoginPage.tsx`, `ForgotPasswordPage.tsx`, `ResetPasswordPage.tsx` — the `"V8"` + `"V88 App"` text
   - `index.html` — `<title>`, `theme-color` meta, `apple-mobile-web-app-title`
   - `public/manifest.webmanifest` — `name`, `short_name`, `theme_color`
   - **App icons** — run `pip3 install --user Pillow && python3 scripts/generate-icons.py` to regenerate every variant (favicon, apple-touch, PWA, maskable) from `v88_context/app-icon.png`. See `skills/app-icons/SKILL.md`.
   - `supabase/functions/bootstrap-admin/index.ts` + `supabase/functions/create-user/index.ts` + `supabase/functions/send-push/index.ts` — `APP_NAME` constant (welcome notifications + push toast title)
   - `supabase/functions/send-email/_branding.ts` — `APP_NAME`, `BRAND_COLOR_HEX`, `LOGO_TEXT`, `SUPPORT_EMAIL` (every transactional email's header, button, footer). Redeploy `send-email` after editing.
   - `public/sw.js` — `APP_NAME` constant (toast title fallback)
   - **Do NOT touch** `src/lib/version.ts` or the "Built with Daia v…" footer on `src/pages/ProfilePage.tsx`. This is the Daia version stamp that downstream upgrade tooling reads — rebrand changes the *app's* identity; the Daia lineage stamp is intentionally separate.
4. **Wait for user approval** before generating any entity code.

---

## What the starter gives you — DO NOT rebuild

The starter ships pre-built. Do not duplicate any of this; extend from it.

**Database (via migrations in `supabase/migrations/`):**
- `profiles` table, `user_role` enum (`admin` / `user`), RLS policies, `on_auth_user_created` trigger, `updated_at` triggers, role helpers (`is_admin()`, `get_user_role()`), self-role-change prevention
- `user_profile_images` storage bucket with RLS
- `notifications` table with RLS + Realtime publication
- `profiles` in the Realtime publication so the AuthContext's disabled-user watch fires
- `push_subscriptions` table — per-device push endpoints, RLS-locked to the owner
- `app_secrets` singleton — VAPID keys, `push_trigger_secret`, `edge_base_url`, `resend_api_key`, `email_from`, `email_reply_to` (default-deny RLS; only service-role + SECURITY DEFINER funcs can read)
- `get_vapid_public_key()` RPC, `notify_push_dispatch` AFTER INSERT trigger (calls send-push via `pg_net`)
- `email_templates` table — content-only rows (subject, intro, optional CTA, outro) keyed by string id; admin RLS for read/write; `is_system=true` rows seeded (`password-reset`, `user-invite`) and protected from deletion by a BEFORE DELETE trigger

**Edge functions (`supabase/functions/`):**
- `bootstrap-admin` — seeds `admin@v88.co.uk` with a welcome notification
- `create-user` — admin creates user; two modes: typed password OR `invite: true` (random placeholder password + invite email via send-email)
- `update-user` — admin updates any field, self-protections for role/disabled
- `delete-user` — hard-delete gated on `disabled=true`, FK check, Storage cleanup (FK check array needs extending as you add tables that reference profiles)
- `send-push` — fans out a notification row to every push endpoint, computes unread count for the OS badge, prunes 404/410 subs
- `send-email` — the ONLY place that talks to Resend. Takes `{ template, to, variables, dry_run? }`, renders an `email_templates` row through the visual shell (`shell.ts`), posts to Resend. Admin or service-role auth.
- `request-password-reset` — anon-callable, anti-enumeration. Generates a recovery link and invokes `send-email` with the `password-reset` template.

**Frontend (`src/`):**
- Contexts: `AuthContext`, `ThemeContext`
- Hooks: `useIsMobile`, `useRealtimeRefresh`, `useNotifications` (incl. OS icon badge sync)
- Push lib: `src/lib/push.ts` — `detectPushSupport`, `subscribePush`, `unsubscribePush`, `isPushSubscribed` (with iOS-installed-PWA gate)
- Layout: `AppLayout`, `AppSidebar` (expand/collapse), `MobileTabBar` (with "More" overflow at > 5 nav items), `AvatarMenu` (profile/theme/sign out + unread notification badge)
- Primitives: `PageHeader`, `Breadcrumb`, `GlassSection`, `GlassModal` (pinned title + scrolling body via `ScrollArea` + pinned footer), `GlassInfoPanel`, `GlassForm` (incl. `EditBtn`/`DeleteBtn`/`AddBtn`), `Avatar`, `ListRow`, `LoadingSkeleton`, `PasswordRules`, `RadialIcon`, `ScrollArea` (overlay-scrollbar primitive for portal/overlay contexts only — never wrap a page or `AppLayout`'s scroll), `StatusBadge`
- Pages: `LoginPage`, `ForgotPasswordPage` (calls `request-password-reset`), `ResetPasswordPage`, `HomePage`, `UsersPage` (Add User modal toggles invite-vs-typed-password), `UserDetailPage` (incl. "Send reset email" button alongside typed-password field), `ProfilePage` (incl. push toggle), `NotificationsPage`, `ComponentShowcase` + `EmailTemplatesShowcase` (maintainer-only at `/dev/components` and `/dev/email-templates` — see below)
- Maintainer surface: `ComponentShowcase` and `EmailTemplatesShowcase` are gated to the seed admin (`admin@v88.co.uk`) via `src/lib/maintainer.ts` + `MaintainerRoute` in `App.tsx`. Entry points are on the Profile page footer — for the seed admin it renders "Built with Daia v… · Email templates", two `<Link>`s; for everyone else (regular admins, standard users) it stays plain text and both routes are URL-blocked by `MaintainerRoute` as defence-in-depth. Both pages render inside `AppLayout` (sidebar + sticky page header), so what they show is what downstream apps actually get. Email templates showcase calls `send-email` with `dry_run: true` so the preview is the same render path production uses.
- Shared nav source: `src/lib/navigation.ts`
- Routing: `src/App.tsx` with `ProtectedRoute` + `AdminRoute`
- PWA: manifest + generic V88 icons (replace during rebrand via `app-icons` skill)
- Service worker: `public/sw.js` — push toasts + OS icon badge. Registered from `main.tsx`.

**Extension points — where to plug your app in:**
- Add nav items to `src/lib/navigation.ts` — they appear in both sidebar and mobile tab bar automatically (mobile overflows to "More" at > 5)
- Add FK dependency checks to `supabase/functions/delete-user/index.ts` as you add tables that reference `profiles`
- Add foundational Realtime subscriptions to `AppLayout`'s `foundationalSubs` array (config, rates, enums — tables whose changes invalidate the whole cache)
- Add new notification triggers on app tables — they ride the existing `notify_push_dispatch` pipeline automatically (in-app + push toast + OS badge with no extra code). See `skills/notifications/SKILL.md`.
- Add new transactional email types by inserting rows into `email_templates` (no edge function redeploy) and calling `send-email` from your trigger/edge-function. See `skills/transactional-email/SKILL.md`.
- Add new migrations under `supabase/migrations/` using timestamped filenames
- Add new pages under `src/pages/` and wire routes in `src/App.tsx`

---

## How V88 Builds Solutions

- **Backend:** Supabase (Postgres, RLS, Edge Functions, Storage, Realtime, Auth)
- **Frontend:** React 19 + Vite + TypeScript + Tailwind + lucide-react (via the starter — don't re-scaffold)
- **Design:** Glassmorphism design system with DM Sans typography (see `ui-conventions` skill)
- **Font:** DM Sans (Google Fonts) — non-negotiable
- **Auth:** Email/password via Supabase Auth. Starter already includes login, forgot-password, reset-password.
- **Builder:** Claude Code (full stack — both frontend and backend)

> ⚠️ **ALL V88 solutions are multi-user. This is non-negotiable.**
>
> Multiple users will be connected simultaneously. Every client must see data changes made by other users instantly — no manual refresh, no stale screens. Supabase Realtime subscriptions are therefore **mandatory on every application table**, wired into React Query cache invalidation via `useRealtimeRefresh`. Not optional.

---

## Skills

Skills live in `skills/`. Each contains a `SKILL.md` with conventions and an optional `references/` folder for deep-dive material loaded on demand. Technique-only skills (those that don't ship code into the starter) keep their copy-on-first-use source files in a `masters/` subfolder — these are the canonical originals to copy into `src/` when adopting the technique.

**Read the relevant `SKILL.md` before generating code.** Match skills to the task:

| Skill | Activate when... |
|-------|------------------|
| `starter-setup` | First time setting up a V88 project from this template — running migrations, deploying edge functions, seeding admin |
| `app-icons` | Generating or refreshing app icons (favicon, apple-touch, PWA, maskable) from a single source PNG. Auto-triggered during rebrand. |
| `notifications` | Working with the notifications subsystem — schema, RLS, adding new notification types, triggers on app tables, push delivery, badge sync, service-worker behaviour |
| `transactional-email` | Sending, designing, editing, or adding transactional email — Resend integration, `email_templates` rows, `send-email` edge function, branding/shell tweaks |
| `supabase-schema` | Writing any SQL, creating tables, designing migrations, naming columns or enums |
| `supabase-auth` | Working with user profiles, roles, RLS policies, auth flows, or any user-management Edge Function |
| `supabase-frontend` | Writing Supabase JS client code, calling Edge Functions, uploading to Storage, subscribing to Realtime, generating types |
| `supabase-postgres-best-practices` | Optimising queries, choosing indexes, reviewing RLS performance, configuring connection pooling, diagnosing slow queries |
| `ui-conventions` | Building any UI component, page, or layout. Read this **entire** skill before writing frontend code |
| `testing` | Writing tests, validating RLS policies, testing migrations, smoke testing |
| `pdf-export` | Adding any PDF export or generation feature |
| `voice-dictation` | Adding speech-to-text input to a form field via the browser's Web Speech API |
| `timeline-gantt` | Building a drag-driven Gantt / resource timeline — bars assigned to resources, drag-to-move dates, drag-to-resize, reorder within a group |

### Skill origins

`starter-setup`, `app-icons`, `notifications`, `transactional-email`, `supabase-schema`, `supabase-auth`, `supabase-frontend`, `ui-conventions`, `testing`, `pdf-export`, `voice-dictation`, and `timeline-gantt` are V88 internal conventions. `supabase-postgres-best-practices` is maintained by Supabase and follows the Agent Skills open standard.

---

## Boundaries

- **Always run `npm run build` before declaring any feature complete** — the dev server hides TypeScript errors that will fail on Vercel. Never use `npx tsc --noEmit` as a substitute.
- **Never modify `SKILL.md` files** — these are maintained upstream by the V88 team. If you're building an app on top of this template, you consume skills, you don't edit them. (If you're the template maintainer editing skills intentionally, see `TEMPLATE.md`.)
- **Never modify `references/` or `masters/` folders inside skills** — `references/` is reference material, `masters/` are the canonical copy-on-first-use source files for technique skills. Both are owned by the skill maintainer.
- **Never make ad-hoc Supabase schema changes** — all changes via timestamped SQL migrations. Never edit schema via the Supabase dashboard.
- **Never use `service_role_key` in frontend code** — it belongs in Edge Functions only.
- **Never disable RLS** on any table.
- **Never use direct colour classes** (`text-white`, `bg-gray-500`) — always use design tokens from `ui-conventions`.
- **Never re-bootstrap the frontend** — the starter ships with `src/`, `package.json`, configs all in place. Do not run `npm create vite@latest` or re-copy reference components.

---

## Code Style

- TypeScript strict mode
- `snake_case` for all database objects (tables, columns, functions, triggers)
- `camelCase` for TypeScript / React code
- `PascalCase` for React components
- 2-space indentation
- Named exports preferred
- All Supabase tables require `id uuid`, `created_at timestamptz`, `updated_at timestamptz`

---
> Source: [jens-v88/kosmos0](https://github.com/jens-v88/kosmos0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
