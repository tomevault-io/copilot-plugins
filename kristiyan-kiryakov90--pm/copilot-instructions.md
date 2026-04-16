## pm

> TaskFlow - Task and project management app for small teams. Multi-page vanilla JavaScript app using Vite + Bootstrap 5 + Supabase.

# CLAUDE.md

## Project Overview

TaskFlow - Task and project management app for small teams. Multi-page vanilla JavaScript app using Vite + Bootstrap 5 + Supabase.

## Critical Commands

**IMPORTANT**: All npm commands run from `frontend/` directory, NOT project root.

```bash
cd frontend
npm install
npm run dev          # Dev server on localhost:5173
npm run build
```

## Architecture

### Multi-Page Application (NOT SPA)
Separate HTML files for each feature:
- `index.html` - Landing + sys_admin bootstrap modal
- `signup.html` - Registration
- `signin.html` - Login
- `dashboard.html` - User overview
- `projects.html` - Project management
- `tasks.html` - Task board
- `admin.html` - Admin panel (team, tags, workflow, company)

### Role-Based Access (3 roles in auth.users metadata)
- `sys_admin` - Created via bootstrap modal (one-time)
- `admin` - Company admin (manages team, tags, workflow)
- `user` - Standard user

Enforce at 3 layers: RLS policies, backend routes, frontend UI.

### Database Key Points
- Multi-tenant via `companies` table
- RLS uses `auth.user_company_id()` for isolation
- Helper functions: `auth.is_company_admin()`, `auth.is_system_admin()`
- Tables: companies, projects, tasks, attachments
- No public users table (data in auth.users metadata)

### Authentication
- Supabase Auth handles everything (no custom auth)
- Standard registration creates `user` role



## Common Pitfalls

1. Running npm from project root instead of `frontend/`
2. Forgetting RLS policies for company isolation
3. Hardcoding company_id instead of using `auth.user_company_id()`
4. Skipping role validation at all three layers
5. Committing `.env` file

## Key Documentation

- `docs/project-technical-summary.md` - Architecture details
- `docs/coding-conventions.md` - Code style
- `docs/security.md` - Security practices

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kristiyan-Kiryakov90) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
