## chci-attendance-system

> You are building **Klinth**, a minimal web-based team attendance system.

# Klinth — Cursor IDE Rules
# Team Attendance System | Vue 3 + Supabase | MVP Beta

## Project Identity
You are building **Klinth**, a minimal web-based team attendance system.
It bridges ZKTeco biometric devices (via USB .dat file import) with a
real-time web dashboard powered by Vue 3 and Supabase.

## Tech Stack
- **Frontend:** Vue 3 (Composition API + <script setup>), Vite
- **State:** Pinia
- **Routing:** Vue Router 4
- **UI:** Tailwind CSS (utility-first, no component framework)
- **Backend:** Supabase (PostgreSQL, Auth, Realtime, RLS)
- **File Parsing:** Custom ZKTeco .dat parser (binary + text)
- **Language:** JavaScript (no TypeScript unless explicitly requested)

## Database (already migrated — do not re-create tables)
Tables in Supabase:
- `userrole` — id, role_name
- `users` — id (uuid, FK auth.users), username, email, fname, minit, lname,
             age, hours_rendered, bio_id, userrole_id, is_active
- `attendance` — attendance_id, user_id, date, time_in, time_out,
                 overtime_in, overtime_out, undertime, status
- `rectification_requests` — id, user_id, attendance_id, rectification_title,
                              description, rectification_type, date, reviewed_by
- `holidays` — id, holiday_name, holiday_type, date

## Supabase Client
- Always import from `@/lib/supabase.js`
- Never expose service role keys in frontend code
- Always handle Supabase errors explicitly: `const { data, error } = await supabase...`

## Code Style
- Use `<script setup>` syntax for all Vue components
- Use `async/await` over `.then()` chains
- Keep components focused — one responsibility per component
- Composables go in `src/composables/use<Name>.js`
- Stores go in `src/stores/<name>.js` using Pinia `defineStore`
- No inline styles — Tailwind classes only
- Prefer `const` over `let`; never use `var`
- Always validate file input before parsing

## Naming Conventions
- Components: PascalCase (e.g. `AttendanceTable.vue`)
- Composables: camelCase prefixed with `use` (e.g. `useZkParser.js`)
- Stores: camelCase (e.g. `useAttendanceStore`)
- Database columns: snake_case (match Supabase schema exactly)
- Frontend variables: camelCase

## ZKTeco .dat File Rules
- There are TWO file types: user data and attendance logs
- User .dat: fixed-width text, fields — PIN, Name, Password, Card, Role
- Attendance .dat: fixed-width text, fields — PIN, Date(YYYY-MM-DD), Time(HH:MM:SS), Verify, In/Out
- Always parse line by line, skip empty or malformed lines silently
- Match parsed users to `users.bio_id` before upserting attendance
- Use `upsert` with `onConflict` to avoid duplicate records
- Never overwrite a `time_out` if it already exists and incoming is `time_in`

## Import Flow (strict order)
1. User uploads .dat file via `DatImportView.vue`
2. `useZkParser.js` reads and classifies the file (user or attendance)
3. Parsed records are previewed in a table before any DB write
4. On confirm, `useImportStore.js` upserts to Supabase in batches of 50
5. Show per-record success/error feedback after import

## Component Structure Reminder
src/
├── lib/supabase.js
├── router/index.js
├── stores/
│   ├── auth.js
│   ├── attendance.js
│   └── import.js
├── composables/
│   ├── useZkParser.js
│   └── useAttendance.js
├── views/
│   ├── auth/LoginView.vue
│   ├── admin/DashboardView.vue
│   ├── admin/ImportView.vue
│   ├── admin/EmployeesView.vue
│   ├── admin/ReportsView.vue
│   └── employee/MyAttendanceView.vue
└── components/
    ├── import/FileDropzone.vue
    ├── import/ParsePreviewTable.vue
    ├── import/ImportSummary.vue
    ├── attendance/AttendanceTable.vue
    ├── attendance/StatusBadge.vue
    └── layout/AppSidebar.vue

## Do Not
- Do not use Options API
- Do not use Vuex (use Pinia only)
- Do not use any CSS framework other than Tailwind
- Do not create new Supabase tables — schema is already set
- Do not store raw passwords anywhere in the frontend
- Do not import .dat files larger than 10MB without chunking
- Do not skip the preview step before DB writes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kliea) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
