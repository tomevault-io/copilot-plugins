## trally

> This file provides AI assistants with a comprehensive overview of the TRally codebase: its architecture, conventions, development workflow, and key rules to follow when making changes.

# CLAUDE.md — TRally Codebase Guide

This file provides AI assistants with a comprehensive overview of the TRally codebase: its architecture, conventions, development workflow, and key rules to follow when making changes.

---

## Project Overview

**TRally** is a discussion-community web application (토론 모임 웹사이트) built for a specific group of members who organize regular debate sessions. It manages:

- Discussion **schedules** (일정)
- Topic **proposals** (주제)
- **User registration** with admin approval workflow
- **Attendance tracking** (planned/in progress)

The UI is primarily in **Korean**.

---

## Architecture

### Backend: ASP.NET Core 10.0

The C# backend (`TRally/Program.cs`) is intentionally minimal. It does **only** three things:

1. Enables Brotli + Gzip response compression for CSS, JS, JSON, and Markdown
2. Registers `.md` files as `text/markdown` MIME type
3. Serves the `wwwroot/` directory as static files with 24-hour cache headers (`Cache-Control: public,max-age=86400`)

There are **no custom API routes**. All data access happens entirely from the browser via the Supabase JavaScript client.

### Frontend: Vanilla JS Single Page Application

`wwwroot/index.html` is the single HTML file. Navigation between views is handled by JS showing/hiding DOM sections — **no client-side router library is used**.

JavaScript is split into four files, loaded in dependency order:

| File | Purpose |
|---|---|
| `js/config.js` | Environment constants (Supabase URL/key, EmailJS keys, admin email) |
| `js/supabase-client.js` | Supabase client init, global state, all DB CRUD functions |
| `js/auth.js` | Login/signup UI, authentication, admin approval, email notifications |
| `js/main.js` | Page navigation, schedule management, topic management, UI utilities |

### Database: Supabase (PostgreSQL)

All data lives in Supabase. Row Level Security (RLS) is enabled on all tables with permissive read policies. The schema is defined in `TRally/wwwroot/supabase-setup.pgsql`.

---

## Directory Structure

```
trally/
├── TRally.sln                          # Visual Studio solution
├── README.md                           # Korean setup guide
├── DOCS.md                             # Detailed JS API reference
├── CLAUDE.md                           # This file
├── .gitignore
└── TRally/
    ├── TRally.csproj                   # .NET 10.0 project file
    ├── Program.cs                      # Server entry point (44 lines)
    ├── appsettings.json                # ASP.NET Core logging config
    ├── import_topics.sql               # Seed data for topics table
    ├── Properties/
    │   └── launchSettings.json         # Dev ports: HTTP 5001, HTTPS 7001
    └── wwwroot/                        # All frontend assets
        ├── index.html                  # SPA entry point
        ├── netlify.toml                # Netlify SPA redirect rules
        ├── supabase-setup.pgsql        # DB schema creation script
        ├── topics.md                   # Markdown list of discussion topics
        ├── css/
        │   └── style.css               # All styles (~1,337 lines)
        └── js/
            ├── config.sample.js        # Template for config.js
            ├── config.js               # Gitignored — created locally
            ├── supabase-client.js      # DB layer + global state
            ├── auth.js                 # Auth logic
            └── main.js                 # Main app logic
```

---

## Database Schema

### `users`
| Column | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `name` | TEXT | Display name |
| `username` | TEXT | Login ID (UNIQUE) |
| `password` | TEXT | Plain text (known limitation — bcrypt planned) |
| `email` | TEXT | |
| `intro` | TEXT | Bio |
| `role` | TEXT | `'admin'` or `'member'` |
| `approved` | BOOLEAN | Admin must approve new accounts |
| `request_date` | TIMESTAMP | When they applied |
| `created_at` | TIMESTAMP | When approved |

### `schedules`
| Column | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `number` | INTEGER | Session number (회차) |
| `date` | TEXT | Discussion date |
| `topic` | TEXT | Topic title |
| `presenter` | TEXT | 발제자 |
| `moderator` | TEXT | 사회자 |
| `location` | TEXT | Venue |
| `guest` | TEXT | Guest speaker |
| `remarks` | TEXT | Additional notes |
| `created_at` | TIMESTAMP | |

### `topics`
| Column | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `author` | TEXT | Topic proposer |
| `topic` | TEXT | Topic title |
| `keywords` | TEXT | Keywords |
| `date` | TEXT | Scheduled discussion date |
| `completed` | BOOLEAN | Whether discussion is done |
| `created_at` | TIMESTAMP | |

### Planned Attendance Tables (not yet fully implemented)
- `attendance_years`, `attendance_members`, `attendance_schedules`, `attendance_records`

---

## Global State (supabase-client.js)

These module-level variables hold app state and are mutated directly by DB functions:

```js
let users = [];          // Approved members
let pendingUsers = [];   // Awaiting admin approval
let currentUser = null;  // Logged-in user object
let schedules = [];      // All schedule entries
let topics = [];         // All topic proposals
```

When modifying data access logic, always keep these in sync after DB mutations.

---

## Key Conventions

### JavaScript Style
- **ES6+ async/await** for all DB calls — no `.then()` chains
- **Error handling**: `try/catch` blocks with `console.warn()` for soft failures; `alert()` for user-facing errors
- **No frameworks**: vanilla DOM manipulation only (`document.getElementById`, `innerHTML`, etc.)
- Modules are not used — all files share the global scope via `<script>` tags in `index.html`

### CSS Conventions
- **Design language**: glassmorphism with blue gradient backgrounds (`#0f172a` → `#1e40af`)
- **Responsive breakpoints**: 1024px (tablet), 768px (mobile), 480px (small mobile)
- All styles are in a single file: `css/style.css`

### Author Ordering
The custom display order for topic authors in `main.js` is:
```
다흰 → 민구 → 아름 → 승종 → 원혁 → 동원 → 기타
```
Preserve this order when modifying author filter/sort logic.

### Admin Role
Users with `role === 'admin'` can:
- Approve/reject/remove members
- Create, edit, delete schedules
- Manage topics

Regular members can only view data and propose topics.

---

## Development Setup

### Prerequisites
- .NET 10.0 SDK
- Visual Studio 2022 (or CLI with `dotnet`)

### First-Time Setup
```bash
# 1. Create config.js from the template
cp TRally/wwwroot/js/config.sample.js TRally/wwwroot/js/config.js

# 2. Edit config.js and fill in real values:
#    - SUPABASE_URL
#    - SUPABASE_ANON_KEY
#    - EMAILJS_PUBLIC_KEY, EMAILJS_SERVICE_ID, EMAILJS_TEMPLATE_ID (optional)
#    - ADMIN_EMAIL

# 3. Run the supabase-setup.pgsql script in Supabase SQL Editor
#    to create all required tables
```

### Running the Server
```bash
cd TRally
dotnet run
```
- HTTP: `http://localhost:5001`
- HTTPS: `https://localhost:7001`

Or in Visual Studio: open `TRally.sln`, press **F5**.

---

## Build & Deployment

### Debug Build
```
Ctrl+Shift+B in Visual Studio
```

### Release Build
Change the dropdown from Debug → Release, then `Ctrl+Shift+B`.

### Publish
Right-click the project in Solution Explorer → Publish → choose target (Folder, Azure, IIS).

### Netlify
`wwwroot/netlify.toml` configures SPA redirect rules so all routes fall back to `index.html`.

---

## Gitignored Files (Never Commit)

| File | Reason |
|---|---|
| `TRally/wwwroot/js/config.js` | Contains live API keys and secrets |
| `.env` | Environment variables |
| `appsettings.Development.json` | Dev-only settings |
| `appsettings.Production.json` | Production secrets |

If you need to reference configuration structure, use `config.sample.js` — it documents all required constants without real values.

---

## Security Considerations

- **Passwords are stored in plain text** in Supabase. This is a known limitation. Do not treat the `password` field as secure. Future plan: bcrypt hashing + JWT.
- **RLS policies** are currently permissive (public read). For production hardening, they should be restricted.
- **Never commit `config.js`** — it holds the Supabase anon key and EmailJS credentials.
- **SQL injection** is not a risk here because all DB access goes through the Supabase JS client using parameterized queries.
- **XSS**: `innerHTML` is used in several places for rendering. Be careful when inserting user-provided content — ensure it is sanitized or escaped.

---

## External Services

| Service | Purpose | Config Key |
|---|---|---|
| [Supabase](https://supabase.com) | PostgreSQL database hosting | `SUPABASE_URL`, `SUPABASE_ANON_KEY` |
| [EmailJS](https://www.emailjs.com) | Email notifications on signup/approval | `EMAILJS_*` constants |
| [SheetJS (XLSX)](https://sheetjs.com) | Excel file import/export for schedules | CDN, no config needed |
| [Marked.js](https://marked.js.org) | Markdown rendering for topics | CDN, no config needed |
| Google Fonts | Poppins + Noto Sans KR | CDN, no config needed |

---

## Planned Features (Roadmap)

- [ ] Password hashing with bcrypt
- [ ] JWT-based authentication
- [ ] Stricter Supabase RLS policies
- [ ] File/image upload functionality
- [ ] Real-time notifications
- [ ] Full attendance tracking system

---

## Key Files Quick Reference

| Task | File(s) to modify |
|---|---|
| Change server middleware or ports | `TRally/Program.cs`, `Properties/launchSettings.json` |
| Add/change a page or layout | `wwwroot/index.html` |
| Change visual design or layout | `wwwroot/css/style.css` |
| Add a DB function or change data model | `wwwroot/js/supabase-client.js` |
| Change login/signup/approval flow | `wwwroot/js/auth.js` |
| Change page navigation or feature logic | `wwwroot/js/main.js` |
| Change/add environment config constants | `wwwroot/js/config.js` (local), `config.sample.js` (template) |
| Recreate DB tables | `wwwroot/supabase-setup.pgsql` |
| Seed topic data | `TRally/import_topics.sql` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ysj-gif)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ysj-gif)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
