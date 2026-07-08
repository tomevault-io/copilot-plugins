## taskflow

> TaskFlow is a Node.js/Express/EJS task management and work allocation platform with two completely isolated sides sharing one database and server:

# TaskFlow — Codebase Reference

## What This App Is

TaskFlow is a Node.js/Express/EJS task management and work allocation platform with two completely isolated sides sharing one database and server:

- **LOCAL side** — internal team/admin interface
- **CLIENT/Portal side** — external client-facing interface

---

## Tech Stack

- **Runtime**: Node.js, Express 4.18
- **Templates**: EJS 3.1 with `express-ejs-layouts`
- **Database**: MySQL2 3.6 (UTC storage, Eastern timezone in app)
- **Real-time**: Socket.IO 4.8 (dual namespace: `/` local, `/portal` client)
- **Auth**: JWT in `tf-token` cookie, bcryptjs, Google OAuth2
- **Email**: Nodemailer + Gmail SMTP (OAuth2 + app-password fallback) — `services/emailService.js`
- **Files**: Multer for uploads, Google Drive API integration
- **Scheduling**: node-cron (`utils/cronJobs.js`)
- **Migrations**: Auto-run on startup via `utils/auto-migrate.js` (71 SQL files in `migrations/`, latest is `072_weekly_roster_2026-07-03.sql`)

---

## Role System

Two distinct role families — middleware uses this to gate routes:

| Family | Roles | Access |
|--------|-------|--------|
| LOCAL | `LOCAL_ADMIN`, `LOCAL_MANAGER`, `LOCAL_USER` | `/` routes only |
| CLIENT | `CLIENT_ADMIN`, `CLIENT_MANAGER`, `CLIENT_USER`, `CLIENT_SALES` | `/portal` routes only |

`middleware/authenticate.js` blocks CLIENT roles from LOCAL routes and redirects them to `/portal`. `portal/middleware/portalOnly.js` enforces CLIENT roles on portal routes.

---

## Directory Structure

```
taskflow/
├── server.js                  # Entry point — Express, Socket.IO, migrations, cron
├── config/
│   ├── db.js                  # MySQL2 pool (UTC, dateStrings: true)
│   ├── socket.js              # Socket.IO init
│   ├── constants.js           # ROLES, PERMISSIONS, TASK_STATUS enums
│   └── multer.js              # File upload config
├── middleware/
│   ├── authenticate.js        # JWT auth + CLIENT role gating
│   ├── authorize.js           # Role/permission checks (requireRoles)
│   ├── spaJson.js             # X-SPA-Request: 1 → return JSON instead of HTML
│   ├── auditLog.js            # Action logging (CREATE, UPDATE, DELETE, etc.)
│   └── botDetect.js
├── routes/                    # LOCAL side routes
│   ├── index.js               # Admin hub, team, reports, queue, workspace
│   ├── tasks.js               # Task CRUD, board, sessions
│   ├── auth.js                # Login, logout, Google OAuth, profile
│   ├── chat.js                # Direct messaging
│   ├── drive.js               # Google Drive
│   └── help.js
├── controllers/               # LOCAL side business logic (~20 controllers)
│   ├── adminHubController.js  # New admin UI pages (1400+ lines)
│   ├── taskController.js      # Task board, sessions, completion (1200+ lines)
│   ├── reportController.js    # Analytics & reports (1100+ lines)
│   ├── devWorkspaceController.js  # Dev workspace (added May 2026)
│   ├── clientQueueController.js   # Client request queue
│   ├── authController.js
│   ├── chatController.js
│   └── ... (userController, leaveController, groupChannelController, etc.)
├── models/                    # LOCAL side DB abstraction (14 models)
│   ├── User.js, Task.js, TaskCompletion.js
│   ├── ClientRequest.js       # Client queue (large — 24K lines)
│   ├── DevProject.js          # Dev workspace projects
│   ├── Chat.js, GroupChannel.js, BridgeChat.js
│   ├── CompOff.js, LeaveRequest.js, Note.js, Reward.js
│   ├── ShiftHistory.js, Notification.js, Roster.js  # Roster.js: weekly weekoff planning (added Jul 2026)
├── services/                  # Shared business logic
│   ├── emailService.js        # Gmail SMTP, 5 email templates, OAuth2
│   ├── taskService.js         # Task helpers
│   ├── googleDriveService.js  # Drive API wrapper
│   ├── authService.js, backupService.js, dashboardService.js, linkUnfurl.js
├── utils/
│   ├── auto-migrate.js        # Runs pending SQL migrations on startup
│   ├── cronJobs.js            # Scheduled tasks
│   ├── timezone.js            # Eastern timezone helpers (DB is UTC)
│   ├── response.js            # ApiResponse.success/error/paginated
│   └── logger.js              # Winston logging
├── views/                     # LOCAL side EJS templates
│   ├── layouts/main.ejs       # Classic UI shell (Bootstrap 5)
│   ├── admin/                 # NEW Admin Hub (dark theme)
│   │   ├── layout.ejs         # Admin hub shell
│   │   ├── dashboard.ejs, work.ejs, queue.ejs, chat.ejs
│   │   ├── workspace.ejs      # Dev workspace (added May 2026)
│   │   ├── users.ejs, attendance.ejs, leaves.ejs, comp-off.ejs, roster.ejs
│   │   ├── my-tasks.ejs, my-attendance.ejs, my-progress.ejs
│   │   ├── taskboard.ejs, all-tasks.ejs, channel.ejs
│   │   └── infoboard.ejs, notes.ejs, drive.ejs, reports.ejs, ...
│   └── classic/               # Classic Bootstrap pages (tasks, users, reports, chat, etc.)
├── portal/                    # CLIENT side — completely separate stack
│   ├── routes/portal.js       # All /portal/* routes
│   ├── controllers/           # 6 portal controllers
│   │   ├── taskController.js, chatController.js, userController.js
│   │   ├── clientRequestController.js, urgentController.js, teamStatusController.js
│   ├── models/                # Portal DB models (portal_tasks table etc.)
│   │   ├── Task.js, Chat.js, UrgentChat.js, Reminder.js, Report.js, CalendarEvent.js
│   ├── middleware/portalOnly.js
│   ├── views/portal/          # Portal EJS templates
│   │   ├── layout.ejs         # Portal shell
│   │   ├── home.ejs, tasks.ejs, requests.ejs, channel.ejs
│   │   ├── calendar.ejs, help.ejs, workspace.ejs
│   │   ├── chat.ejs, notes.ejs, team-status.ejs, users.ejs, reports.ejs
│   ├── public/
│   │   ├── portal.js          # Client-side SPA JS (large, ~106K lines)
│   │   └── portal.css
│   └── socket/portalSocket.js # /portal Socket.IO namespace
├── public/                    # LOCAL side static assets
├── migrations/                # 59 sequential SQL files (auto-run on startup)
├── uploads/                   # File storage (tasks/, portal/, bridge/, urgent/)
└── frontend/                  # React SPA (separate, Vite-built — experimental)
```

---

## LOCAL Side — Two Parallel UIs

There are two co-existing UIs for the LOCAL side:

### 1. Classic UI (`views/layouts/main.ejs`)
- Bootstrap 5 full-page layout
- Left sidebar nav + right Group Channel panel + floating bridge-chat widget
- Used by all LOCAL roles currently

### 2. Admin Hub (`views/admin/layout.ejs`)
- Dark theme, VSCode-style — opt-in via "✨ Try New Admin UI" button (LOCAL_ADMIN/MANAGER only)
- 240px fixed sidebar + topbar with queue badge, notification bell, channel drawer, client messages drawer
- **CSS variables**: `--adm-bg` (#1a1a1a), `--adm-surface` (#242424), `--adm-border` (#383838), `--adm-text`, `--adm-accent` (#00d4ff cyan)
- No Bootstrap — all custom CSS with `--adm-*` vars
- Gradually replacing classic UI; new features go here first

---

## Key Architectural Patterns

### SPA-style partial refresh
Portal and admin hub pages use `X-SPA-Request: 1` header. `middleware/spaJson.js` intercepts `res.render()` and returns only the data JSON instead of full HTML. This avoids full-page reloads.

### Migrations
`utils/auto-migrate.js` tracks which `.sql` files in `migrations/` have been applied (stored in a `migrations` table) and runs new ones on every server start. To add a migration: create `migrations/073_description.sql`.

### Timezone
All datetimes stored in DB as UTC. App runs on Eastern timezone. Use helpers in `utils/timezone.js` — never do raw `new Date()` for display.

### Audit logging
`middleware/auditLog.js` — wrap mutations with `auditLog(action, resourceType)` to log user actions automatically.

### Email service
`services/emailService.js` — live, uses Gmail SMTP (`servicea@123cfc.com`). Has 5 built templates. Call via `emailService.send(template, to, data)`. Not yet wired into all controllers.

### Socket.IO namespaces
- `/` — LOCAL side events (task updates, notifications, bridge chat)
- `/portal` — CLIENT side events (portal chat, urgent messages, request status)

---

## Features Overview

| Feature | LOCAL side | CLIENT/Portal side |
|---------|-----------|-------------------|
| Task board | `views/admin/taskboard.ejs`, `views/admin/work.ejs` | `portal/views/portal/tasks.ejs` |
| Messaging (direct) | `routes/chat.js` | `portal/controllers/chatController.js` |
| Group channel | `views/admin/channel.ejs` | `portal/views/portal/channel.ejs` |
| Urgent chat | Bridge chat widget in layout | `portal/controllers/urgentController.js` |
| Client request queue | `views/admin/queue.ejs` (approve/reject) | `portal/views/portal/requests.ejs` (submit) |
| Dev workspace | `views/admin/workspace.ejs` | `portal/views/portal/workspace.ejs` |
| Reports/analytics | `views/admin/reports.ejs`, `reportController.js` | `portal/views/portal/reports.ejs` |
| Leave management | `views/admin/leaves.ejs` | — |
| Attendance/shifts | `views/admin/attendance.ejs` | — |
| Comp-off credits | `models/CompOff.js` | — |
| Weekly roster (weekoff planning) | `views/admin/roster.ejs`, `models/Roster.js` — admin/manager plan next week's weekoffs, employees request a day | Read-only (Team Status shows the roster-resolved weekoff, no CLIENT edit access) |
| Info board | `views/admin/infoboard.ejs` | — |
| Calendar | — | `portal/views/portal/calendar.ejs` |
| Team status | — | `portal/views/portal/team-status.ejs` |
| Google Drive | `routes/drive.js` | — |
| Email notifications | `services/emailService.js` + `utils/cronJobs.js` | Triggered by queue actions |

---

## Database Notes

- Main task table: `tasks` + `task_instances` (for recurring tasks)
- Portal tasks are in a **separate** `portal_tasks` table — not shared with LOCAL tasks
- Client requests: `client_requests` table (managed by `models/ClientRequest.js`)
- Dev workspace: `dev_projects` + related tables (added May 2026, migrations 056–058)
- Comp-off cancel/revoke: `comp_off_credits.status` now has `revoked` value (migration 059, June 2026)
- Weekly roster: `weekly_rosters` (per user/week weekoff assignment) + `roster_requests` (employee-submitted day requests) tables (migration 072, July 2026). `models/Roster.js` resolves the effective weekoff for a date — published roster row overrides `users.weekly_off_day`, which remains the fallback. Every weekoff check app-wide (comp-off trigger, attendance calendars, taskboard, live status, reports, portal Team Status, cron reminders) reads through this resolver except the two raw-SQL recurring-task-scheduling filters in `models/Task.js`, which still use the static column.
- Migrations table: tracks applied SQL files by filename

---

## Admin Email
- App admin/notification email: `servicea@123cfc.com`
- Dev/owner email: `sunilgupta@123cfc.com`

---
> Source: [sunilgupta-arch/taskflow](https://github.com/sunilgupta-arch/taskflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
