## teamnotes

> **IMPORTANT:** When committing changes that add features, modify architecture, change the database schema, or alter the project structure, you MUST also update this file (CLAUDE.md) and README.md to reflect those changes before committing. Keep both files accurate and current.

# TeamNotes - Claude Code Project Context

## Documentation Maintenance Rule
**IMPORTANT:** When committing changes that add features, modify architecture, change the database schema, or alter the project structure, you MUST also update this file (CLAUDE.md) and README.md to reflect those changes before committing. Keep both files accurate and current.

## Quick Start
```bash
npm install    # Install dependencies
npm run dev    # Dev server at http://localhost:3000
npm run build  # Production build to ./dist
```

## What This App Is
A team collaboration app where **leaders** create team goals, assign reports, and manage monthly recurring tasks, while **members** manage personal tasks, notes, and personal goals that can link to team goals. Built with React 18 + Vite + Supabase (PostgreSQL) + Tailwind CSS. Deployed to GitHub Pages via GitHub Actions on push to `master`.

## Architecture Overview

### Stack
- **Frontend:** React 18, React Router 6, Tailwind CSS 3, Vite 5
- **Backend:** Supabase (PostgreSQL + Auth + Realtime subscriptions)
- **Deployment:** GitHub Pages via GitHub Actions (`master` branch)

### State Management (React Context + Custom Hooks)
```
BrowserRouter (basename="/teamnotes")
  AuthProvider (user session, profile, auth methods)
    TeamProvider (teams, goals, members, categories)
      Pages → use hooks: useTasks(), useNotes(), usePersonalGoals(), useReports()
```

### Key Directories
```
src/
  components/
    auth/       - LoginForm, SignupForm, ProtectedRoute
    common/     - Layout, Button, Modal, LoadingSpinner
    dashboard/  - LeaderDashboard, MemberDashboard, MemberViewDashboard (inline edit + full TaskEditor modal),
                  GoalCard, GoalFormModal, CategoryManagementModal, ReportsDashboard
    tasks/      - TasksList (kanban+list views with 4 columns: To Do/In Progress/On Hold/Done, standard/monthly toggle), TaskCard, TaskEditor
    notes/      - NotesList, NoteCard, NoteEditor
    goals/      - PersonalGoalsList, PersonalGoalCard, PersonalGoalEditor
    team/       - TeamSettings, MemberList, InviteMember
    settings/   - AccountSettings, SecuritySettings, PreferenceSettings
  contexts/
    AuthContext.jsx  - Auth state, signIn/signUp/signOut, profile management
    TeamContext.jsx  - Teams, goals, members, categories, realtime subscriptions
  hooks/
    useTasks.js          - Tasks CRUD, multi-assignee, personal goal links, monthly tasks
    useNotes.js          - Notes CRUD, task linking
    usePersonalGoals.js  - Personal goals CRUD, year filtering, task/goal links (batch queries)
    useReports.js        - Reports CRUD (leader-only feature)
    useTeam.js           - Wrapper for TeamContext
  pages/
    Login.jsx, Signup.jsx, Dashboard.jsx, MyTasks.jsx,
    MyNotes.jsx, PersonalGoals.jsx, TeamSettings.jsx, Settings.jsx
  lib/
    supabase.js  - Supabase client initialization
```

### Routes (App.jsx)
- `/login`, `/signup` - Public
- `/dashboard` - LeaderDashboard or MemberDashboard based on role
- `/tasks`, `/notes`, `/personal-goals` - Member workspace
- `/team` - Leader-only team management
- `/settings` - Account/security/preferences
- `*` - Redirects to `/dashboard`

## Database Schema (Supabase)

### Core Tables
- **profiles** - User accounts (id, email, display_name, password)
- **teams** - Team containers (name, leader_id)
- **team_members** - Membership junction (team_id, user_id, role: leader|member)
- **goals** - Team goals (title, description, status, due_date, category_id, sort_order, notes, show_notes)
- **tasks** - User tasks (title, description, status: todo|in_progress|done|on_hold, due_date, linked_goal_id, shared_to_dashboard, notes, sort_order, completed_at, is_monthly, monthly_source_id, monthly_month)
- **notes** - User notes (title, content, linked_goal_id, shared_to_dashboard)
- **personal_goals** - Personal goals (title, description, status, year, sort_order)
- **categories** - Goal categories (name, color, sort_order)
- **invitations** - Email invites (email, team_id, status: pending|accepted)
- **reports** - Leader-assigned report items (title, team_id, assigned_user_id, created_by)

### Junction Tables (Many-to-Many)
- **task_assignees** (task_id, user_id)
- **goal_members** (goal_id, user_id)
- **task_personal_goal_links** (task_id, personal_goal_id)
- **goal_personal_goal_links** (goal_id, personal_goal_id)

### SQL Reference
- `supabase-schema.sql` - Base schema reference (migration files removed after being applied)

### Monthly Tasks
- Tasks with `is_monthly=true` and `monthly_source_id=null` are **templates**
- Templates auto-create monthly **instances** (`monthly_source_id` points to template, `monthly_month` = 'YYYY-MM')
- `ensureMonthlyTasks()` runs on load to create current month's instances
- Monthly instances appear on the "By Member" dashboard only within 7 days of month-end
- Dashboard has All/Standard/Monthly filter toggle

### Reports Feature
- Leader-only feature, 3rd tab on Leader Dashboard (Goals | By Member | Reports)
- Simple items: title assigned to a team member
- Grouped by member in the dashboard view (same visual pattern as "By Member")
- Inline create/edit/delete

## Code Patterns & Conventions

### Data Fetching Pattern (Custom Hooks)
```javascript
// 1. Main query with joins
const { data } = await supabase.from('table').select('*, related (id, title)')...
// 2. Batch fetch junction table records
const [assignees, links] = await Promise.all([...])
// 3. Build maps, attach to records
// 4. Set state
```

### Many-to-Many Updates (Replace-All Pattern)
```javascript
// Delete existing junction records, then insert new ones
await supabase.from('junction').delete().eq('main_id', id)
await supabase.from('junction').insert(newIds.map(...))
```

### Form Handling
- Controlled components with useState
- `saving` boolean for button disable/loading text
- `error` state for inline error display
- `e.preventDefault()` on submit

### Styling
- Tailwind CSS with custom component layer (`.btn`, `.card`, `.badge`, `.input`)
- Dark mode via `class` strategy (`dark:` prefix)
- Primary color: blue scale (primary-50 through primary-900)
- Responsive: `md:grid-cols-2`, `md:grid-cols-3` patterns

### Roles
- **Leader** (`isLeader` from TeamContext): Can manage goals, categories, members, reports, see all shared items
- **Member**: Can manage own tasks/notes/personal goals, share items to dashboard

## Known Issues & Technical Debt
- Auth uses plaintext passwords (not Supabase Auth) - stored in profiles table
- Supabase credentials hardcoded in `src/lib/supabase.js` (anon key is public, but still)
- No test coverage
- `test-queries.mjs` at project root is a debug script (not production code)

## Deployment
- Push to `master` triggers `.github/workflows/deploy.yml`
- Builds with Node 20, injects Supabase vars from GitHub Secrets
- Deploys to GitHub Pages at `/teamnotes/` base path

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nate91117)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nate91117)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
