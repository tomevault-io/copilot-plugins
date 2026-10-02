## shift-management

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

飲食店（やよい軒 JR森ノ宮店）向けのLINE連携シフト管理Webアプリ。スタッフはLINE LIFF経由でログインし希望シフトを提出、管理者はガントチャート形式で確定シフトを作成してPDFで公開する。

Monorepo with two independently deployed halves:
- `frontend/` — React 19 + Vite SPA, deployed to Vercel
- `backend/` — Spring Boot 3.5 (Java 21) REST API, deployed to Render (via Docker), backed by PostgreSQL (Neon)

## Commands

### Frontend (`frontend/`)
```bash
npm install       # install deps
npm run dev       # start Vite dev server (localhost:5173)
npm run build     # production build
npm run lint      # eslint .
npm run preview   # preview production build
```
No test runner is configured for the frontend.

### Backend (`backend/`)
```bash
./mvnw spring-boot:run     # run the API (or run ShiftManagementApplication.java directly from an IDE)
./mvnw test                 # run all tests
./mvnw test -Dtest=ClassName#methodName   # run a single test
./mvnw package -DskipTests  # build the jar (this is what Dockerfile does)
```
There is essentially one placeholder test (`ShiftManagementApplicationTests`, just a context-load check) — no meaningful test suite exists yet.

Backend requires environment variables (see `.env` in `backend/`, gitignored): `DB_URL`, `DB_USERNAME`, `DB_PASSWARD` (note: typo is intentional, matches `application.properties`), `CLOUDINARY_CLOUD_NAME/API_KEY/API_SECRET`, `LINE_CHANNEL_ACCESS_TOKEN`, `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_BUCKET`.

Frontend requires `VITE_LIFF_ID` (LINE LIFF app ID) in `frontend/.env`. Note the API base URL is currently **hardcoded** in `frontend/src/api/api.js` (points at the Render production URL), not read from an env var, despite `VITE_API_BASE_URL` being documented in the README — check this file before assuming env-based config works.

## Architecture

### Backend: standard layered Spring Boot
`controller/` (REST endpoints, `@RequestMapping("/api/...")`) → `service/` (business logic, `@Transactional` where multi-row writes happen) → `repository/` (Spring Data JPA) → `entity/` (JPA entities, one table each). `dto/` holds request/response records used at the controller boundary — entities are never returned directly from controllers.

Key domain entities (see README ER diagram for full field list):
- `User` — STAFF/ADMIN role, HALL/KITCHEN position, `sortOrder` for admin-configured display order, `lineUserId` as the LINE-auth identity.
- `ShiftRequest` — a staff member's per-day availability submission for a given `periodId`.
- `ConfirmedShift` — the admin-finalized per-day assignment for a given `periodId`.
- `SubmissonPeriod` (sic — misspelled entity/table name, kept as-is in DB) — largely superseded by `SubmissionPeriodService`'s rolling calculation (see below); do not assume the table is the source of truth for the current period.
- `ShiftPdf` — one row per published period, storing the Supabase Storage URL of the confirmed-shift PDF.

There is no authentication/authorization layer on the backend — role checks (`STAFF` vs `ADMIN`) happen client-side only, based on the `role` field returned at login and cached in `localStorage`. CORS is locked to two explicit origins in `CorsConfig` (`localhost:5173` and the Vercel prod URL) — update both when adding a new frontend origin.

**Rolling submission period**: `SubmissionPeriodService.getCurrentPeriod()` computes the "current" 14-day shift period and its deadline purely from a hardcoded `BASE_DEADLINE` constant and today's date (JST) — it does **not** read `SubmissonPeriod` from the DB. When changing period/deadline logic, this service is the actual source of truth, not the entity/repository of the same name.

**Idempotent submission pattern**: `ShiftRequestService.submit()` upserts via `findByUserIdAndWorkDate()` (update if exists, insert if not) inside one `@Transactional` batch — this is what prevents duplicate submissions and keeps DB round-trips low. `ConfirmedShiftService.submit()` instead does delete-then-reinsert per `periodId` (admin "temporary save" is a full overwrite, not a merge). Follow whichever pattern matches the existing method when extending these services.

**External integrations**, each isolated in its own service:
- `SupabaseStorageService` — uploads confirmed-shift PDFs to Supabase Storage (bucket path = `{periodStart}.pdf`, upsert on conflict); returns a public URL. Despite the `ShiftPdf` entity/field names referencing "Cloudinary," PDF storage has migrated to Supabase — Cloudinary (`CloudinaryConfig`, `cloudinary-http45` dependency) is still wired up but PDF upload no longer uses it.
- `LineNotificationService` — pushes LINE messages via the Messaging API; fails soft (logs and swallows errors) if `LINE_CHANNEL_ACCESS_TOKEN` is unset or the call fails, since notification failures should never block the primary action.
- `SubmissionReminderScheduler` — a daily `@Scheduled` cron job (9:00 JST) that pushes a LINE reminder to any user who hasn't submitted for the current period's start date, but only on the day before the deadline.

All server-side "what day is it" logic uses `ZoneId.of("Asia/Tokyo")` explicitly, since the Render host runs in UTC — preserve this when adding new date-sensitive code.

### Frontend: page-per-route, no global state library
Routing is a flat list in `App.jsx` (React Router v6, no nested/protected routes — access control is just conditional rendering based on `loginUser.role`). Each page under `src/pages/` independently:
1. Calls `initLiff()` (`src/liff/liff.js`) to get the LINE profile and log in/register via `POST /api/users/login`.
2. Reads/writes `loginUser` from `localStorage` (no context provider or state management library).
3. Calls the backend directly through the shared `axios` instance in `src/api/api.js`.

`Layout.jsx` provides the shared header/footer chrome and derives the page title from a path→title lookup table (`PAGE_TITLES`) — add new routes there too when adding a page.

`AdminConfirmedShiftCreate.jsx` (~1300 lines) is the largest and most complex page: a gantt-chart shift editor with distinct PC (drag-based, `@dnd-kit`) and mobile (tap-to-open-panel, <768px breakpoint) interaction modes, plus `html2canvas`/`jsPDF` for client-side PDF export before upload.

Styling is a single global `src/index.css` (no CSS modules/styled-components/Tailwind).

## Conventions

- Indentation is **tabs** everywhere (enforced by `.editorconfig`), 4-width for Java, 2-width for JS/JSON/CSS/HTML/MD.
- CRLF line endings, UTF-8, final newline required (`.editorconfig`).
- Frontend formatting: Prettier with `useTabs: true` (`.prettierrc`).
- Many recent commits/comments are in Japanese, matching the target users and existing codebase — match this when editing existing files with Japanese comments.
- Comments in this codebase tend to explain *why* a non-obvious tradeoff was made (e.g. batching writes into one transaction, delete-then-reinsert vs. upsert, JST timezone handling) — follow that style rather than describing *what* the code does.

---
> Source: [onaga-daiki-ocs2026/shift-management](https://github.com/onaga-daiki-ocs2026/shift-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
