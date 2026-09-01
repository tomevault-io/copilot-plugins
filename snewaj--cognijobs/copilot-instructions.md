## cognijobs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Current status (read this first)

**The app has been built.** This repository started as a pure documentation/handover package for "Bdjobs", an
employment platform — but the build described below has since been completed end to end and the app was renamed
**Bdjobs → CogniJobs** (commit `60bf04c`). There is now real source code:

- **`backend/`** — the .NET modular monolith (12 bounded-context modules), built from `Handover_Packages/BC-*.md`.
- **`frontend/`** — the Next.js 4-portal app, built from `Handover_Packages/FA-*.md`.
- **`infra/`** — AWS EC2 deployment scripts (Docker Compose + nginx on a single host).

All 18 units of the build pipeline described below are `done` (`.orchestration/STATUS.md`), with full backend/
frontend test suites green as of the last pass (see `HANDOVER.md`). Current work has shifted to deployment, not
app-building — **`HANDOVER.md` (repo root) is the up-to-date session handover; read it first for what's actually in
flight.** `BUILD_REPORT.md` is the full historical build log if you need narrative detail on how a given module/portal
was built or what bugs were fixed along the way.

The rest of this file (below) describes the **original spec** that drove the build. It's kept as design-rationale
reference — useful if you're extending the app and want to understand *why* something was built a certain way — but
it is not a from-scratch build brief anymore. Note also that `Handover_Packages/`, `Stories/`, and `SRS_Sections/`
are **not tracked in git** (see root `.gitignore`) — they exist on disk locally but aren't part of the shipped repo.
The spec still says "Bdjobs" throughout (expected — it predates the rename and wasn't updated, since it's frozen
reference material, not shipped documentation).

If asked to **extend or modify the app**, read the existing code in `backend/`/`frontend/` first — that's the
ground truth now, not the spec. Cross-reference the relevant `Handover_Packages/BC-*.md` or `FA-*.md` file only when
you need the original design rationale (contracts, acceptance criteria) behind existing behavior.

## Original spec structure (historical — see above)

There are three top-level folders that made up the original handover package:

- **`Handover_Packages/`** — the build briefs.
- **`Stories/`** — supporting artifacts: user stories, DDD strategic design (context map, event catalog), and UI traceability (screen catalog, story↔screen map).
- **`SRS_Sections/`** — the underlying Software Requirements Specification, split into one file per section (also an Obsidian vault — `SRS_Sections/.obsidian/`). This is the source requirements; `Handover_Packages/` and `Stories/` are derived from it for build purposes.

## Entry points (read in this order) — for understanding the original spec/design rationale

1. **`Handover_Packages/BUILD_ORCHESTRATION.md`** — the master orchestrator brief. Describes the program as an **18-step sequential, single-worker-at-a-time** pipeline (backend foundation → 12 backend modules in dependency waves → frontend foundation → 4 frontend portals), with a durable checkpoint/resume protocol under `/.orchestration/` so work survives context limits. **All 18 units are now `done` per `.orchestration/STATUS.md`** — this checkpoint/resume protocol is no longer live; treat `STATUS.md` as a historical record of build order, not a resume point, unless the app itself needs a from-scratch rebuild. (Originally 19 steps/5 portals — step 19/FA-4 Partner was dropped when FA-4 merged into FA-3; see below.)
   - **Default to this sequential mode, not parallel multi-agent fan-out.** §1/§4 are explicit: "one worker at a time", "No two workers run at once" — spawn one worker for the next unit, wait for it to finish and merge, checkpoint, then advance. The "running modules/portals in parallel" sections in the two track guides (§5 of each) are an **optional**, opt-in alternative for fanning out within a wave/phase — they are not what the master orchestrator does by default, and should only be used if explicitly requested.
2. **`Handover_Packages/BACKEND_BUILD_INSTRUCTIONS.md`** — entry point for backend work (the 12-module modular monolith).
3. **`Handover_Packages/CLAUDE_CODE_BUILD_INSTRUCTIONS.md`** — entry point for frontend work (the 4-portal web app).
4. **`Handover_Packages/00-Shared-Foundations.md`** — cross-cutting backend brief: the (initially blank) Target Stack declaration, neutral type/notation vocabulary, 5-layer module structure, shared-kernel building blocks (`Entity<Id>`, `AggregateRoot<Id>`, `Result`/`Result<T>`, `Error`), outbox/inbox conventions, testing strategy. Every `BC-*` package assumes this.
5. **`Handover_Packages/00-Frontend-Foundations.md`** — cross-cutting frontend brief: target stack, design system, app shell, routing/guards, API client, `Error.Code` → UI mapping, i18n, accessibility, testing. Every `FA-*` package assumes this.
6. **`Stories/Context_Map.md`** — DDD strategic design showing how the 12 bounded contexts relate (OHS/PL, ACL, Customer/Supplier, Partnership, Conformist). Determines backend build/parallelization order.
7. **`Stories/Event_Catalog.md`** — the ~70 integration events that are the published language between backend modules.

## Architecture this spec describes

**Backend** — one deployable modular monolith, 12 bounded-context modules, each with its own Domain/Application/Infrastructure/API/Contracts layers and its own DB schema (`Handover_Packages/BC-01-*.md` … `BC-12-*.md`). Modules talk **only** through their `Contracts` layer and integration events via outbox/inbox — never by reading another module's tables. Two CORE subdomains: BC-7 Recommendation Engine (AI matching) and BC-10 Reporting (LMIS analytics); the rest (IAM, profiles, postings, applications, search, sync, notification, taxonomy, content) are generic/supporting and feed the cores.

**Frontend** — one Next.js program, **four** portals on one shared design system/API client (`Handover_Packages/FA-01-*.md`, `FA-02-*.md`, `FA-03-*.md`, `FA-05-*.md`): Job Seeker (`/`), Employer (`/employer`), Admin (`/admin`), Public/Guest (`/`). Portals integrate with the backend only through the HTTP contracts documented in each `BC-*` package's §12. **`FA-04-Partner.md` is no longer a fifth portal** — the Partner/Integration console is admin-operated, not partner self-service, and its 4 screens were absorbed into FA-3 Admin's Integration oversight cluster (see "Resolved decisions" below). The file is kept for screen/domain reference only.

**Target stacks are declared.** Both Target Stack blocks are filled in — treat them as authoritative, don't re-derive a stack from general defaults:
- **Backend** (`00-Shared-Foundations.md` §1): C# 13 (.NET 10), ASP.NET Core 10 (Minimal APIs + MVC/Web API), EF Core 10 + Dapper (perf-critical reads), PostgreSQL 18 (Npgsql), MediatR, FluentValidation, xUnit + FluentAssertions + NSubstitute, Testcontainers for .NET + PostgreSQL, **SignalR** (real-time push hub, §6.6).
- **Frontend** (`00-Frontend-Foundations.md` §1, also restated in `CLAUDE_CODE_BUILD_INSTRUCTIONS.md` §3): TypeScript, React 19, Next.js 16 (App Router), TanStack Query, Zustand, Tailwind CSS v4 + CSS-variable design tokens, Radix UI + React Aria, React Hook Form + Zod, react-i18next, Axios (+ OpenAPI-generated client), Vitest + React Testing Library + MSW, Playwright, @axe-core/playwright + eslint-plugin-jsx-a11y, **Recharts** (charting, for FA-3 Admin Analytics), **@microsoft/signalr** (real-time client).
- Every chart still needs its own data-table/text alternative per the a11y rules below; Recharts doesn't provide that automatically.

## Hard rules from the handover packages

These apply once implementation starts (see each guide's own "Golden rules" section for the full list):

- **Never invent a backend endpoint or integration event.** Every frontend call maps to a row in a `BC-* §12` table; every cross-module fact goes through the frozen `Event_Catalog.md` contracts. Missing functionality is an open question to raise, not something to assume.
- **Contracts are frozen after Phase A ("B0"/"F0") and read-only thereafter.** Backend module internals and frontend portals build against the frozen `§12` API contracts + Event Catalog; changes go through versioned contract governance, not ad hoc edits.
- **No module reads another module's tables; no FK crosses a module boundary.** Cross-module references are plain `uuid` columns.
- **Backend is authoritative.** The frontend mirrors validation for UX only and always defers to the server's `Result`/`Error`; branch on `Error.Code`, never on HTTP status or message text.
- **Domain layers are persistence-ignorant and exception-light** — expected failures return `Result`/`Result<T>`, not exceptions.
- **WCAG 2.1 AA + full English/Arabic RTL i18n on every screen** — no hard-coded UI strings, no color-only status.
- **Every acceptance criterion in a package's §11/§13/§14 must map to ≥1 passing test** before that unit is "done".

## Resolved decisions (formerly open questions)

Every cross-program blocking/open question originally flagged in the spec has been decided by the maintainer and written back into the relevant packages. Don't re-litigate these or treat them as still-open — each package's own §14/appendix carries the full resolution and rationale; this is the index:

- **FA-4 Partner → merged into FA-3, admin-operated.** BC-8 stays admin-only; no partner self-service. `FA-04-Partner.md` header note, [[FA-03-Admin]] §14 (new **S-ADM-30** screen absorbs the former FA-4 screens), `CLAUDE_CODE_BUILD_INSTRUCTIONS.md`/`BUILD_ORCHESTRATION.md` updated (4 portals, 18 steps).
- **FA-3 role→cluster matrix** — defined in [[FA-03-Admin]] §14 (SystemAdministrator/MoLAdministrator: full access; others scoped per-cluster).
- **Real-time vs polling → push.** SignalR (backend hub) + `@microsoft/signalr` (frontend client), added to both Target Stack blocks; mechanics in `00-Shared-Foundations.md` §6.6. Drives notifications, FA-3's integration monitor/live dashboards, and BC-10's exact session tracking (new `UserLoggedOutIntegrationEvent`, BC-1 §8.1).
- **Feature flags → role-only**, no separate flag system (`00-Frontend-Foundations.md` appendix).
- **BC-2 "shortlists" vs BC-7 "talent-pools" → kept as two distinct concepts.** BC-2's event renamed `CandidateAddedToShortlist`; BC-7's `CandidateSavedToTalentPool` promoted to a real integration event (BC-2/BC-7 §8, [[Event_Catalog]], BC-9/BC-10 updated).
- **Taxonomy source for the job-posting form → BC-11, general master/lookup data, not skills-only.** BC-11 gained a new non-admin `GET /api/taxonomies/Skills/terms` endpoint (its `TaxonomyApi` is in-process-only, not HTTP-reachable — `BC-11-Administrators-Configuration.md` §12), added to FA-2's `consumes_backend` and `CLAUDE_CODE_BUILD_INSTRUCTIONS.md` §2's bundle table. Contract type/work format/education level are **not** BC-11 taxonomies — they're closed enums on BC-4's `JobPosting` aggregate, shipped as static frontend option lists. **Superseded by CR-US-3.2.1-01-01**: BC-11 is no longer skills-only — it gained a second taxonomy kind, `Countries` (`GET /api/taxonomies/Countries/terms`), seeded with ISO 3166-1 alpha-2 codes, backing BC-04's mandatory Location field. `ITaxonomyApi` gained generic, kind-parameterized `MapTerm`/`IsValidCode` counterparts to the Skills-specific `MapSkill`/`IsValidSkillCode`, establishing the pattern for future master/lookup lists beyond just Skills and Countries.
- **Offers in BC-5 → first-class `Offered`/`OfferRescinded` stages** with dedicated `POST /api/applications/{id}/offer[/rescind]` endpoints (`BC-05-Job-Application.md` §6.2/§10/§12), replacing the generic stage-transition workaround.
- **Public job-detail projection → dedicated endpoint.** `GET /api/public/job-postings/{id}` → `PublicJobPostingDto` on BC-4 (`BC-04-Job-Postings.md` §12); FA-1/FA-5 wired to it.
- **Auto-close ambiguity → kept current default** (always-automatic expiry; `AutoCloseEnabled` stays a stored preference only).
- **AccountDeactivationCascade saga → owned by BC-1**, as an in-module process manager (`DeactivationCascadeRun`, BC-1 §8.3) rather than a 13th bounded context.
- **News article by-slug route, SSR/SEO strategy, featured-jobs source** (FA-5 §14) — BC-12 gained an `Article.Slug` + `GET /api/content/news/{slug}`; per-route SSR/SSG+ISR split defined; featured jobs resolved via a new BC-4 `IsFeatured` admin flag flowing into BC-6's search filters (`FeaturedToggle` added to FA-3's S-ADM-03).
- **My Applications (FA-1 S-SEEK-13) column scope** — defined, backed by BC-5's existing `GET /api/applications` (FA-1 §14).
- **BC-10 catalog gaps → resolved.** `UserLoggedOutIntegrationEvent` added (BC-1 §8.1); `PerformanceAlertRaisedIntegrationEvent` added to [[Event_Catalog]]; `SystemMetricSampled` deliberately stays a non-catalog external telemetry feed (confirmed, not changed).

A few **pedagogical "teaching notes & open questions"** remain in the `BC-*` appendices (e.g. BC-11 taxonomy Published-Language-vs-Shared-Kernel, resume-parsing BC-3-vs-BC-7 ownership, granular-vs-unified event design) — these are intentional discussion prompts where the package already states and justifies its chosen design; they are not blockers and weren't part of this resolution pass.

## Conventions used throughout the spec files

- Handover packages are written **language/framework-neutral**: types like `uuid`, `decimal`, `datetime`, contract shapes like `MethodName(param: type) -> ReturnType`, and roles like "the chosen ORM" are specifications to translate into the declared stack, not literal code.
- `BC-NN-*.md` = backend bounded-context package. `FA-0N-*.md` = frontend portal package. `US-X.Y.Z-NN-*.md` (in `Stories/`) = individual user story, traceable via `Stories/BC_Mapping.md` (story↔backend module) and `Stories/Story_Screen_Map.md` (story↔screen).
- Cross-references inside the Markdown use Obsidian-style `[[WikiLinks]]` (e.g. `[[BC_Mapping]]`) — these resolve to other files in the same folder by basename.

---
> Source: [snewaj/CogniJobs](https://github.com/snewaj/CogniJobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
