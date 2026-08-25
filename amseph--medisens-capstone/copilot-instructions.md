## medisens-capstone

> **MediSens** is a secure, role-based Rural Health Unit information system and Electronic Medical Record prototype for Malvar RHU.

# CLAUDE.md

## Project

**MediSens** is a secure, role-based Rural Health Unit information system and Electronic Medical Record prototype for Malvar RHU.

Primary goals:

- Improve patient-record retrieval and continuity of care.
- Support role-specific RHU workflows.
- Reduce duplicate encoding and manual reporting work.
- Protect sensitive health information through layered authorization and privacy controls.
- Provide useful clinical, geographic, and operational analytics without exposing patient-level data.

This is a capstone system. Treat it like a real healthcare application, not a generic demo dashboard.

---

## How Claude Code Should Work

### Grounding and uncertainty

Before changing code:

1. Inspect the real implementation.
2. Verify actual files, routes, components, types, tables, columns, RPCs, role checks, and existing patterns.
3. Briefly summarize the verified architecture relevant to the task.
4. List any uncertainty, conflict, or missing information.
5. Ask a focused question before editing when a requirement is ambiguous, risky, or has multiple valid interpretations.

Do not hallucinate:

- Files or folders
- Routes
- Components
- Hooks
- Database columns
- RPC names
- Role permissions
- Existing functionality
- Framework behavior

Treat filenames and implementation suggestions in prompts as guidance until verified in the repository.

If the requested behavior conflicts with the current architecture, explain the conflict before changing anything.

Do not claim something was preserved unless it was inspected or tested.

### Scope discipline

- Implement only the requested task.
- Do not silently add features.
- Do not perform unrelated cleanup.
- Do not redesign unrelated modules.
- Do not broaden access or permissions.
- Do not rewrite working architecture merely because another pattern is cleaner.
- Prefer the smallest safe change that fits existing conventions.
- Reuse existing components and logic instead of duplicating them.

Ask before making changes that affect:

- Authentication
- Authorization
- Role permissions
- Routes
- Supabase schema
- RLS
- RPCs
- Edge Functions
- Shared application shells
- Cross-role workflows

---

## Division of Work

Claude Code is primarily used for:

- UI and UX implementation
- Frontend refactoring
- Responsive layouts
- Component architecture
- Visual hierarchy
- Design-system consistency
- Accessibility
- Frontend performance
- Analytics page organization

Codex is primarily used for:

- Supabase migrations
- SQL
- RLS
- Security hardening
- Edge Functions
- Complex backend logic
- Database seeding
- High-risk authorization changes

Do not modify backend security or database behavior unless the task explicitly assigns it to Claude Code.

---

## Roles

Supported application roles include:

- `admin`
- `doctor`
- `nurse`
- `bhw`
- `midwives`
- `pharmacist`
- `labaratory`

Important: the stored laboratory role string is intentionally spelled:

```text
labaratory
```

Do not “correct” this value to `laboratory` in authorization logic, database queries, or role comparisons unless a dedicated migration is explicitly requested.

---

## MediSens UI work

Before implementing any visible UI change, read:

- `docs/design/SKILL-UI.md`
- `docs/design/UI-CLINICAL-PATTERNS.md` when the task involves patient or clinical interfaces
- `docs/design/PWA-OFFLINE-TARGET.md` only when the task explicitly involves offline or synchronization behavior

Before modifying visible UI, inspect:

- `docs/design/medisens-ui-reference.png`

This image is the **canonical visual source of truth** for MediSens.

The reference defines the application's visual language, not its functionality. Adapt its hierarchy, spacing, typography, proportions, component quality, card treatment, restrained claymorphism, layout rhythm, and visual density to MediSens.

Do not copy its:

- Branding
- Logo
- Financial terminology
- Dashboard content
- Charts
- Widgets
- Product-specific workflows

Do not replace existing layouts merely to resemble the reference. Adapt the reference's design language to the verified MediSens structure and workflows.

Preserve existing:

- Workflows
- Features
- Permissions
- Role guards
- RPCs
- Database behavior
- Validation
- Clinical rules
- Business logic

When redesigning:

- Prefer updating shared components and centralized tokens before modifying individual pages.
- Reuse existing shared components instead of creating duplicate visual variants.
- Check every role shell that consumes a changed shared component.
- Keep module-specific changes scoped to the requested module.
- Do not alter shared application shells unless the task explicitly includes them.
- Do not implement future PWA or offline behavior merely because it appears in a planning document.

If the reference conflicts with accessibility, clinical safety, confirmed MediSens requirements, or existing business logic, preserve MediSens behavior and adapt only the presentation.

---

## Product Workflows

MediSens includes role-based workflows for:

- Patient registration and patient records
- Initial consultation
- Vital signs
- Doctor consultation
- Follow-ups
- Laboratory requests and results
- Electronic prescriptions and dispensing
- Vaccination and FHSIS records
- Maternal care
- Archive review
- Audit logging
- Clinical and geographic analytics

Preserve current workflows and ownership boundaries unless the request explicitly changes them.

Never assume that UI visibility alone is sufficient authorization.

---

## Security Rules

Security is layered across the UI, application services, Supabase functions, and database policies.

Preserve:

- Role-based access control
- Row Level Security
- Server-side validation
- Fixed `search_path` for `SECURITY DEFINER` functions
- Restricted execute grants
- Revoked `public` and `anon` access where applicable
- Audit logging
- Valid workflow-state transitions
- Patient-consent restrictions
- Archive-field protections
- Aggregate-only analytics
- Small-count suppression for sensitive analytics

Never:

- Disable RLS for convenience
- Weaken authorization to fix a UI issue
- Put service-role keys in frontend code
- Expose patient-level data through analytics
- Return patient names, IDs, addresses, contact details, notes, diagnoses, findings, or prescription contents from aggregate analytics RPCs
- Infer privileged access from navigation visibility
- Store secrets in source code
- Modify migration history after it has been applied, unless explicitly instructed

When an applied migration needs correction, create a new corrective migration.

---

## Analytics Architecture

Keep one **Analytics** item in the sidebar.

The Analytics workspace is organized into internal views:

- Clinical Analytics
- Geographic Analytics
- Staff Operations

### Access

Doctor:

- Clinical Analytics
- Geographic Analytics
- Staff Operations

Midwife:

- Clinical Analytics
- Geographic Analytics
- No Staff Operations access

Other roles:

- Preserve existing restrictions.
- Do not grant new access without an explicit requirement.

Doctors must retain full access to Geographic Analytics.

Staff Operations must remain Doctor-only unless a future requirement explicitly changes it.

### Analytics navigation

The existing outer application navigation may use hash-based sections.

For Analytics sub-tabs:

- Use a `view` query parameter when supported by the verified implementation.
- Preserve existing outer hash behavior.
- Support refresh and browser back/forward.
- Invalid or unauthorized views must fall back to an authorized view.
- Do not introduce a new global router for this refactor unless explicitly requested.

Expected conceptual states:

```text
?view=clinical
?view=geographic
?view=staff
```

Verify the actual routing implementation before editing.

### Shared Analytics chrome

The period preset/custom date filter and global loading, updating, offline, and error states are shared above the Analytics tab bar.

Clinical KPI cards belong only inside Clinical Analytics.

Clinical Analytics contains:

- Clinical KPI strip
- Clinical charts and summaries
- Detailed Records
- Clinical analytics note card

Geographic Analytics contains:

- Malvar barangay map
- Metric selector
- Barangay ranking
- Geographic coverage summary
- Hover tooltip
- Selected-barangay summary
- Barangay aggregate drill-down
- Geographic heatmaps

Staff Operations contains only real, supported aggregate operational data.

Do not add fake staff metrics, fake charts, or unsupported scores.

### Geographic Analytics semantics

Preserve:

- GeoJSON-based Malvar map
- Map, ranking, and drill-down synchronization
- Metric switching
- Selected barangay state
- Keyboard accessibility
- Tooltip bounds
- Aggregate-only output
- Privacy-safe small-count suppression
- Date-filter semantics
- Zero-value barangay handling

Registered-patient metrics may use all-time active patient semantics, while activity metrics may use the selected date period. Labels must clearly state the scope.

Do not let hover movement trigger backend drill-down requests.

---

## Staff Operations

Use the label **Staff Activity & Service Operations** or **Staff Operations** rather than implying a complete employee-performance score.

Do not create:

- Overall performance scores
- Cross-role leaderboards
- Unfair comparisons between incompatible roles
- Patient-level staff drill-downs
- Rankings based on weak or missing attribution

Separate:

- Productivity metrics
- Completion metrics
- Turnaround metrics

Display reliability or attribution limitations when the backend provides them.

---

## UI and UX

Follow `docs/design/SKILL-UI.md`.

Design direction:

- Clinical and trustworthy
- Clean hierarchy
- Modern but restrained
- No generic AI-SaaS appearance
- No unnecessary gradients
- No excessive cards
- No decorative charts without decision value
- No emojis inside the interface

### Shared components first

When redesigning:

- Update shared components before applying one-off page styling.
- Extend an existing component when it already solves most of the requirement.
- Avoid role-specific visual forks for controls that should look and behave consistently.
- Check all consumers before changing a shared component.
- Preserve established keyboard, focus, loading, responsive, and mobile-drawer behavior.

### Shared component library

Treat the following as shared across role shells unless the verified implementation documents a necessary exception:

- Application shell
- Sidebar
- Header
- Mobile navigation drawer
- Page layout
- Page header
- Buttons
- Inputs
- Selects
- Search fields
- Filters
- Cards and summary surfaces
- Tables and list rows
- Status badges
- Tabs
- Dialogs
- Drawers and slide-overs
- Toasts and alerts
- Skeletons
- Empty states
- Error states
- Loading and updating indicators

Do not create a new visual variant merely because a component appears in a different role.

### Design tokens

`docs/design/SKILL-UI.md` is authoritative for the visual token system.

All new or revised:

- Colors
- Spacing
- Typography
- Border radii
- Borders
- Shadows
- Elevation
- Focus treatments
- Motion values

must come from centralized design tokens.

Reuse the existing token source. If a required token does not exist, add it to the existing theme or token file rather than introducing a parallel system or a one-off value inside a component.

Preserve semantic color meaning:

- Green for success, online, and completed states
- Amber for warning, pending, due, and offline-related attention states
- Red for errors, destructive actions, urgent states, and critical clinical values
- Navy/blue for brand, selection, focus, navigation, and informational states

Do not recolor medical or workflow statuses in ways that make them ambiguous.

### Typography

Use centralized design tokens. The current blue-indigo foundation is:

```css
--brand-primary: #5A81FA;
--brand-active: #2B318A;
--text-primary: #1F1F1F;
--text-secondary: #6A6E83;
--text-muted: #A8B1CE;
--brand-accent-surface: #D1DDFF;
--brand-soft-surface: #F2F3FF;
--page-background: #F7F8FC;
--surface: #FFFFFF;
```

Preserve semantic colors:

- Green for success, online, and completed states
- Amber for warning, pending, and offline states
- Red for error, destructive, and urgent states
- Blue/indigo for brand, selection, focus, and information

Do not recolor medical or workflow statuses in ways that make them ambiguous.

### Typography

- Use the centralized typography system.
- Use the existing licensed font stack.
- Do not download or embed unlicensed font files.
- Keep body text readable.
- Avoid excessive bold text and letter spacing.
- Maintain clear hierarchy for page titles, section titles, card titles, labels, metadata, metrics, and table text.
- Preserve practical print typography for prescriptions and clinical documents.

### Sidebar

Sidebar groups must be role-aware and based on visible routes.

Possible categories include:

- Overview
- Patient Care
- Clinical Workflow
- Diagnostics
- Pharmacy Operations
- Maternal & Community Care
- Insights
- Records & Governance
- Administration

Rules:

- Do not show empty category headings.
- Do not expose unauthorized routes.
- Keep headings non-interactive and visually secondary.
- Preserve active-route behavior.
- Preserve collapsed and mobile behavior.
- Preserve the logo area and user-profile area.

---

## Loading and Refresh Behavior

Use these rules consistently:

### Initial load

- Show content-shaped skeletons that match the final layout.
- Keep skeletons inside the affected section.
- Do not blank the whole application shell.

### Background refresh

- Preserve existing content.
- Show a subtle inline updating state.
- Do not replace content with a full skeleton.
- Do not trigger full-page reloads for filters, tabs, or refreshes.

### Errors

- Localize errors to the failing section whenever possible.
- One failed optional analytics request must not blank the entire Analytics workspace.
- Do not silently convert real errors into zero values.
- Preserve previous successful data during transient refresh failures when safe.

### Requests

- Deduplicate identical requests.
- Ignore stale responses after a newer request starts.
- Avoid requests caused only by pointer movement.
- Use stable request keys for selection and date-based queries.

---

## Empty States

Empty states must explain why no data is shown.

Differentiate between:

- No records exist
- No records match the current filter
- Data is unavailable because a request failed
- Data is intentionally suppressed for privacy
- The feature is not implemented yet

Do not display misleading zeroes when the correct state is unavailable or suppressed.

Do not use fake content to fill whitespace.

---

## Forms, Tables, and Clinical UI

- Keep visible labels.
- Do not rely only on placeholders.
- Preserve validation and error messaging.
- Maintain adequate touch targets.
- Keep dense tables readable and scannable.
- Use drawers or slide-overs only when they fit the existing workflow.
- Do not change clinical form meaning during visual refactors.
- Preserve print layouts and document outputs.

---

## Responsive Behavior

Optimize for:

- Desktop
- Tablet
- Mobile PWA across common phone widths

Rules:

- Avoid horizontal scrolling.
- Avoid squeezing desktop dashboards into narrow screens.
- Stack complex analytics sections logically.
- Keep map interactions usable on touch devices.
- Preserve safe-area spacing.
- Maintain readable typography and touch targets.
- Do not crop device-height content behind fixed navigation.

---

## Database and Supabase Changes

When a task explicitly requires backend work:

1. Inspect actual migrations and schema first.
2. Verify table and column names.
3. Never guess a database field.
4. Preserve existing authorization helpers.
5. Use one narrowly scoped migration.
6. Use `CREATE OR REPLACE FUNCTION` only when appropriate.
7. Fix ambiguous SQL references with explicit aliases.
8. Apply consistent period semantics:

```text
p_from <= timestamp < p_to_exclusive
```

9. Exclude null or negative turnaround intervals.
10. Return aggregate-only analytics data.
11. Keep `public` and `anon` execution revoked where required.
12. Do not apply or push migrations unless explicitly asked.

If generated Supabase types are missing, state that limitation and verify against current migrations or the live schema before implementation.

---

## Database Seeding

Defense/demo data must be:

- Fully synthetic
- Internally consistent
- Deterministic when possible
- Re-runnable
- Cleanly removable
- Distributed across realistic date ranges
- Distributed across Malvar barangays
- Linked across complete workflows
- Free of real patient information

Do not:

- Use real patient records
- Disable security permanently
- Bypass constraints casually
- Seed trigger-generated tables directly without verifying behavior
- Create random disconnected rows merely to fill charts

Create backup and rollback instructions before destructive cleanup.

---

## Code Quality

- Follow existing project patterns.
- Prefer typed interfaces.
- Avoid `any` unless unavoidable and explained.
- Keep components focused.
- Extract shared components only when reuse is real.
- Avoid premature abstractions.
- Avoid duplicated business logic.
- Keep styling centralized.
- Use semantic tokens instead of scattered hardcoded values.
- Preserve accessibility.
- Do not remove comments that explain security or unusual workflow decisions.
- Do not add verbose comments that restate obvious code.

---

## Git and Verification

Do not commit or push unless explicitly instructed.

Before reporting completion:

```bash
npm.cmd run build
git diff --check
```

Also perform targeted verification appropriate to the change.

For UI changes, verify:

- Desktop
- Tablet
- Mobile
- Keyboard navigation
- Hover and focus states
- Empty states
- Loading states
- Error states
- Role-specific visibility

For analytics changes, verify:

- Doctor access
- Midwife access where allowed
- Unauthorized-role denial
- Date filters
- Zero totals
- Privacy suppression
- No duplicate requests
- No console errors
- No patient-level data exposure

Report:

- What changed
- Files changed
- What was preserved
- Tests run
- Remaining risks or uncertainties
- Whether a migration or deployment action is required

Do not hide warnings or failed checks.

---

## Final Rule

MediSens handles sensitive health information.

When choosing between a faster change and a safer, better-grounded change, choose the safer one.

When uncertain, stop and ask.

---
> Source: [amseph/MEDISENS-Capstone-](https://github.com/amseph/MEDISENS-Capstone-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
