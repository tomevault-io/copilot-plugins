## bni-champion-chapter-tw

> This file guides Codex agents working in the `take-seat` project.

# AGENTS.md

This file guides Codex agents working in the `take-seat` project.

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Project Identity

`take-seat` is a Next.js 16 App Router project for BNI 長冠軍分會 seat planning.

The current product is a weekly seating workspace:

- `/` is a public entry page.
- `/seats` is the main drag-and-drop seating workspace.
- `/seats/print` renders the print/PDF view from saved workspace state.
- Current data is static TypeScript seed data plus browser `localStorage`.
- The next product direction is a fuller event website with live seat map, attendance presence, member voting, and evidence-backed operating workflows.

Do not treat this repository as `2026-nuvaclub`. That repository is a reference for agent workflow, batch tasks, and evidence reports only.

## Commands

Use package scripts that exist in this repository:

```bash
pnpm run dev
pnpm run build
pnpm run lint
git diff --check
```

No test framework is currently configured. If you need automated coverage, add it intentionally and document the reason.

## Required Reading Order

Before changing code, read in this order:

1. This `AGENTS.md`.
2. Relevant Next.js 16 docs under `node_modules/next/dist/docs/`.
3. `docs/00_manual-and-index/MAN-001_document-index.md`.
4. Relevant product, architecture, feature, plan, and acceptance docs under `docs/`.
5. Relevant source files.
6. `package.json` scripts.

For App Router work, usually read:

- `node_modules/next/dist/docs/01-app/01-getting-started/02-project-structure.md`
- `node_modules/next/dist/docs/01-app/01-getting-started/05-server-and-client-components.md`
- `node_modules/next/dist/docs/01-app/02-guides/data-security.md`
- Route/API/caching docs that match the specific change.

## Current Architecture

Current important files:

- `src/app/page.tsx` - home entry page.
- `src/app/seats/page.tsx` - server page that loads current week seed data and mounts the workspace.
- `src/app/seats/print/page.tsx` - print route.
- `src/components/SeatingArranger.tsx` - main client workspace, dnd, validation summary, CSV/PDF actions.
- `src/components/RuleEditor.tsx` - rule/roster editing panel.
- `src/types/seating.ts` - domain-facing seating types.
- `src/lib/seating-week.ts` - current week selection.
- `src/lib/layout-*.ts` - weekly seat roster/layout seeds.
- `src/lib/seating-storage.ts` - `SeatingWorkspaceRepository` and localStorage implementation.
- `src/lib/seating-validation.ts` - seating rule validation.
- `src/lib/seating-export.ts` - CSV export helpers.

Current data boundary:

```txt
seed roster/layout -> /seats server page -> client workspace -> localStorage draft -> CSV/print export
```

Future data boundary should evolve toward:

```txt
domain rules -> application DTO/services -> repository port -> storage/API implementation -> route/component UI
```

## Engineering Rules

- Keep source changes small and tied to a documented task, bug, acceptance case, or explicit user request.
- Preserve unrelated worktree changes. This repo currently has many uncommitted files; do not revert them.
- Prefer TypeScript domain types over ad hoc objects.
- Keep browser-only APIs such as `window`, `localStorage`, drag-and-drop sensors, and print actions inside Client Components or client-side repositories.
- Keep pages/layouts as Server Components unless interactivity is required.
- Do not pass private attendee/member data into public Client Components. Create explicit DTOs for public, member, staff, and admin views.
- Use `lucide-react` icons for UI controls when an icon exists.
- Avoid production database mutations unless the user explicitly approves the target environment and mutation scope.
- Never store secrets, raw cookies, tokens, QR confirmation codes, or credentials in docs, reports, screenshots, or committed files.

## Documentation Structure

Use this docs shape for new work:

```txt
docs/
  00_manual-and-index/       index, usage manuals, onboarding
  01_product-requirements/   PRD files
  02_architecture-and-rules/ architecture, contracts, security rules
  03_feature-reference/      implemented feature references
  05_execution-plans/        batch plans and implementation plans
  08_acceptance-and-qa/      acceptance criteria and QA reports
  2_agent-input/             agent loop instructions and generated evidence
```

Existing legacy docs in the root of `docs/` are still valid references. Do not move them casually; moving docs should be its own documented cleanup batch.

## Acceptance-Driven Workflow

Use this loop for meaningful product work:

```txt
Read product/architecture/acceptance docs
-> inspect current code behavior
-> choose one batch or narrow task
-> design minimal contracts and UI states
-> implement
-> validate with scripts/browser proof where relevant
-> save evidence/report under docs/2_agent-input/generated/agent-loop/reports/
```

Do not mark a task as passed without evidence. If a task cannot be automated, mark it `MANUAL_REQUIRED` and explain the exact manual check.

## Live Seat Map Product Direction

The next larger product arc is documented in:

- `docs/01_product-requirements/PRD-001_live-seat-map-attendance-voting.md`
- `docs/02_architecture-and-rules/ARC-001_live-seat-map-attendance-voting-architecture.md`
- `docs/05_execution-plans/PLN-001_live-seat-map-attendance-voting-batch-plan.md`
- `docs/08_acceptance-and-qa/ACC-001_live-seat-map-attendance-voting-acceptance.md`

Core privacy rule:

- Public view: anonymous seat/attendance statistics only.
- Member view: own seat and allowed public profile cards only.
- Staff view: check-in workflow and scoped attendance summary.
- Admin view: full operating cockpit, exports, and audit trail.

Email, confirmation codes, raw user IDs, private notes, and full attendee lists must never appear in public DTOs.

## Active Product Architecture Correction

The current seat planning UX is shifting from a single weekly editor into a one-to-many model:

```txt
Seat Template / Source Plan
-> many event date seat maps
-> each event date has public page, poll, results, and exports
```

Read this architecture note before touching `/seats`, `/admin`, public event pages, or poll creation:

- `docs/02_architecture-and-rules/ARC-005_one-to-many-seat-template-event-page-architecture.md`

Important correction:

- Do not keep adding event/date creation controls inside `src/components/SeatingArranger.tsx`.
- `/seats` should become the index/list/create surface.
- `/seats/[weekId]` should become the focused editor for one event date.
- Event operations such as publish, poll setup, close, results, and exports should be scoped to one event, preferably under `/admin/events/[weekId]` or a clearly separated event operations tab.

### Implementable Batch Tasks: Seat One-To-Many Refactor

Work these batches in order. Do not skip ahead unless the user explicitly redirects.

#### SEAT-IA-001 - Decision Lock

Status: Implemented

Goal: confirm and preserve the one-to-many target before changing UI again.

Tasks:

- [x] Read `ARC-005_one-to-many-seat-template-event-page-architecture.md`.
- [x] Confirm target editor route is `/seats/[weekId]`.
- [x] Confirm `/seats` should no longer default into the editor once the index exists.
- [x] Confirm whether event creation requires Google login only or admin password as well.

Decisions:

- Editor route is `/seats/[weekId]`.
- `/seats` is the one-to-many seat map index and create surface.
- Event creation is not Google-only; Google session or admin access can authorize create operations.
- Same-day multiple meetings are not supported; `MeetingSession.weekId` remains unique by date.
- Named reusable seat templates are in scope now.

Stop/report if:

- The user wants a different route shape.
- The user wants named reusable templates now, not later.
- The user wants multiple meetings on the same date.

Evidence:

- Update an agent-loop report with decisions and changed files.

#### SEAT-IA-002 - Build `/seats` Index

Status: Implemented

Goal: make `/seats` a seat map management index instead of a single editor.

Tasks:

- [x] Create a server-side list loader for existing `MeetingSession` records with latest `SeatMap`, public status, poll count, open poll count, and last updated time.
- [x] Build `/seats` as a quiet operational index page.
- [x] Show event rows with date, title, seat count, assigned count, public status, poll status, and actions.
- [x] Add an index-level create/duplicate entry point.
- [x] Link each row to `/seats/[weekId]`.
- [x] Keep Google sign-in visible for actions that require persistence.

Acceptance:

- `/seats` communicates that there can be many event seat maps.
- No drag/drop editor appears on the index page.
- The user can choose an event before editing.

Validation:

```bash
pnpm run lint
pnpm run build
git diff --check
```

Manual evidence:

- Browser check `/seats` desktop and mobile.

#### SEAT-IA-003 - Move Editor To `/seats/[weekId]`

Status: Implemented

Goal: isolate drag/drop editing to one event date.

Tasks:

- [x] Add `src/app/seats/[weekId]/page.tsx`.
- [x] Move current editor behavior from `/seats` into `/seats/[weekId]`.
- [x] Load DB seat map by `weekId`; fall back only with an explicit not-found/create path.
- [x] Keep `SeatingArranger` focused on editing, saving, local draft, validation, CSV, and PDF.
- [x] Remove inline `新座位表日期` and `建立指定日期模板` from `SeatingArranger`.
- [x] Redirect `/seats?weekId=YYYY-MM-DD` to `/seats/[weekId]` for compatibility.

Acceptance:

- `/seats/[weekId]` edits exactly one event date.
- LocalStorage remains isolated by `weekId`.
- Save/sync still writes `PATCH /api/seats/[weekId]`.

Validation:

```bash
pnpm run lint
pnpm run build
git diff --check
```

Manual evidence:

- Browser check editing and saving on one existing `weekId`.

#### SEAT-IA-004 - Create/Duplicate Contract

Status: Implemented

Goal: replace the current ambiguous template creation endpoint with an explicit duplication flow.

Tasks:

- [x] Design API contract for creating an event seat map from a source.
- [x] Prefer `POST /api/seats` for create or `POST /api/seats/[weekId]/duplicate` for source-based duplication.
- [x] Support source choices:
  - latest event
  - selected event
  - base TypeScript seed template
- [x] Add duplicate-date confirmation or clear overwrite/version behavior.
- [x] Deprecate or remove `POST /api/seats/templates` once the replacement is live.
- [x] Record operation log for event seat map creation.

Acceptance:

- Admin can choose target date and source before creation.
- The created event opens in `/seats/[weekId]`.
- Existing event dates are not silently overwritten.

Validation:

```bash
pnpm run lint
pnpm run build
git diff --check
```

Manual evidence:

- Create one test date only after explicit user approval, then verify it appears in `/seats`.

#### SEAT-IA-005 - Event Operations Placement

Status: Implemented

Goal: make publish/poll/result/export controls clearly belong to one event.

Tasks:

- [x] Create or plan `/admin/events/[weekId]` as the event operations cockpit.
- [x] Move publish controls out of generic `/seats` index/editor if they clutter editing.
- [x] Move poll controls out of generic `/admin` global login record page.
- [x] Keep `/admin` as global admin dashboard and login record view.
- [x] Link from `/seats` event row to event operations.

Acceptance:

- Poll creation always happens after selecting one event date.
- Voting results and exports are event-specific.
- Public/admin DTO separation remains intact.

Validation:

```bash
pnpm run lint
pnpm run build
git diff --check
```

Manual evidence:

- Browser check selected event operations route.

#### SEAT-IA-006 - Print Route Cleanup

Status: Ready after SEAT-IA-003

Goal: align print/PDF with one event date.

Tasks:

- [ ] Add `/seats/[weekId]/print` or explicitly keep `/seats/print` as state-based print only.
- [ ] Ensure print title/date match the selected `weekId`.
- [ ] Prevent printing stale `print-state` from a different event without visible warning.

Acceptance:

- Printed PDF cannot accidentally show the wrong event date.

Validation:

```bash
pnpm run lint
pnpm run build
git diff --check
```

#### SEAT-IA-007 - Named Reusable Templates

Status: Implemented

Goal: allow admins to preserve a reusable named seat template and use it when creating future event seat maps.

Tasks:

- [x] Add a `SeatTemplate` Prisma model for reusable named snapshots.
- [x] Add template list/create repository functions.
- [x] Add `GET /api/seat-templates` and `POST /api/seat-templates`.
- [x] Let `/admin/events/[weekId]` save the selected event's latest seat map as a named template.
- [x] Let `/seats` create a new event from a named template.
- [x] Keep one event per date by preserving `MeetingSession.weekId` uniqueness.
- [x] Allow event creation through Google session or admin access, not Google-only.

Acceptance:

- Admin can save an event seat map as a reusable template.
- Admin can choose that template when creating a new event date.
- Existing one-date-one-event behavior remains enforced.

Validation:

```bash
pnpm run prisma:generate
pnpm run prisma:validate
pnpm run lint
pnpm run build
git diff --check
```

Manual evidence:

- Run `pnpm run prisma:push` before using the new `SeatTemplate` collection.
- Browser check saving a template from `/admin/events/[weekId]`.
- Browser check creating a new date from the saved template on `/seats`.

#### SEAT-IA-008 - Public Attendance Check-In

Status: Implemented

Goal: allow public event page users to mark assigned seats as arrived and keep public pages synchronized.

Tasks:

- [x] Add public attendance status to `PublicSeatDTO`.
- [x] Add `PATCH /api/public/events/[slug]/attendance`.
- [x] Update the current seat assignment status between `assigned` and `checked_in`.
- [x] Record an operation log for public attendance changes.
- [x] Add public page seat-card buttons for `抵達` and `取消抵達`.
- [x] Poll the public event DTO so other open public pages receive the update.
- [x] Keep public voting independent from attendance status.
- [x] Render the public main seat map with at least five columns.

Acceptance:

- Public page shows current checked-in count.
- Clicking a seat's attendance button updates the seat card and summary.
- Other public pages see the change after polling or refresh.
- Public poll selection does not require the candidate or voter to be marked arrived.

Validation:

```bash
pnpm run prisma:validate
pnpm run lint
pnpm run build
git diff --check
```

Manual evidence:

- Browser check `/w/[slug]`, click one occupied seat's `抵達`, and confirm another browser/tab updates within the polling window.

## Agent Loop

For long-running work, use:

- `docs/2_agent-input/AGENTS.md`
- `docs/2_agent-input/generated/agent-loop/README.md`
- `docs/2_agent-input/generated/agent-loop/development-strategy.md`
- `docs/2_agent-input/generated/agent-loop/loop-state.json`
- `docs/2_agent-input/generated/agent-loop/report-template.md`

Stop and report when:

- The user says `STOP_AND_REPORT`, `暫停回報`, or similar.
- A visible page slice is ready for review.
- A cross-page flow is connected enough for review.
- A product/security/data decision is needed.
- Validation fails for a real issue.

Reports must include changed files, visible routes, contracts, validation commands, evidence paths, and the recommended next task.

## Current Batch Status

Initial planning/docs batch:

- [x] Establish docs folder structure for take-seat.
- [x] Create product, architecture, execution, acceptance, and agent-loop docs.
- [x] Rewrite root `AGENTS.md` for take-seat.
- [ ] Implement live event website routes.
- [ ] Implement persistent seat map domain/repository.
- [ ] Implement attendance presence states.
- [ ] Implement member voting.
- [ ] Add browser/API evidence for public/member/staff/admin boundaries.

One-to-many seat page refactor:

- [x] Research one-to-many seat template/event page architecture.
- [x] SEAT-IA-001 Decision Lock.
- [x] SEAT-IA-002 Build `/seats` index.
- [x] SEAT-IA-003 Move editor to `/seats/[weekId]`.
- [x] SEAT-IA-004 Create/duplicate contract.
- [x] SEAT-IA-005 Event operations placement.
- [ ] SEAT-IA-006 Print route cleanup.
- [x] SEAT-IA-007 Named reusable templates.
- [x] SEAT-IA-008 Public attendance check-in.

---
> Source: [OliverTai688/bni-champion-chapter-tw](https://github.com/OliverTai688/bni-champion-chapter-tw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
