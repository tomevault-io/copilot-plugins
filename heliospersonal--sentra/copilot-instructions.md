## sentra

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Sentra — a mobile **personal health-management app** ("personal lab-result archive + checkup planner") for the EU + Ukraine markets, mobile-first. The full design/spec chain (SDLC 01→03.5) is done for all 7 MVP features, and the **MVP is now implemented**: the `server/` .NET modular monolith (F0–F6 backends behind ports-and-adapters, EF+Postgres, integration tests) and the `mobile/` Flutter app (all six MVP feature UIs) are built. Product documentation lives under `docs/` alongside the Penpot screen designs. The raw founder note `ideas.md` lives at the repo root; the maintained product index is [[BACKLOG]] under `docs/`. See [[READING-GUIDE]] for how the docs chain together, and the **Working notes** section below for how to build/run/test the code.

**This is a clean greenfield.** Nothing is shipped, there are no users, no data, and no migrations. Don't worry about historical data, backward compatibility, deprecation paths, or migrating anything — design for the right end state directly. If a file or decision turns out wrong, just change or delete it. (The only "history" is `ideas.md`, kept purely as the origin note — not a constraint.)

**Solo project — one person wears every hat.** The owner/PM/designer/developer is the same person: **Viacheslav Melnichenko**, a .NET backend developer (~8 years, microservices). So:
- Role labels in docs (`owner: PM`, `owner: Tech Lead`, `owner: UX`, "medical advisor") are **hats, not people** — they all currently resolve to Viacheslav. They mark *which mindset* a task needs, not a hand-off to someone else. Don't treat them as a team or invent stakeholders.
- Backend/architecture calls can lean on real .NET + microservices expertise — be concrete and senior there, not hand-holding.
- There is no separate budget, legal team, or medical staff yet. When a task implies one (lawyer sign-off, medical advisor), treat it as "Viacheslav to source later" — a flagged dependency, not a blocker to design around. Prefer approaches a solo founder can actually execute now (curate from public sources + disclaimer) over ones that need hiring.

## Documentation language policy (IMPORTANT)

**English is the single source of truth for all documentation.** Every document is authored and maintained in English.

- `<doc>.md` — **English. The source of truth.** When anything changes, change this file.
- `<doc>.uk.md` — **Ukrainian. Reference/comprehension copy only.** It exists so Ukrainian-speaking stakeholders can read along. It is **never** the authority and **never** the basis for downstream work (PRD, architecture, ADR, code).

Rules:
- **Ukrainian (`.uk.md`) lives only under `docs/`.** No other directory (`src/`, `app/`, config, scripts, tests, etc.) may contain Ukrainian — code, comments, identifiers, commit-adjacent files, and all non-`docs/` content are English-only. Ukrainian is a documentation-reading aid, nothing more.
- When you create or edit any English doc under `docs/`, update its `.uk.md` sibling in the same commit so they do not drift. If you cannot, note the `.uk.md` as stale rather than leaving a silent mismatch.
- **Exception — deep-technical artifacts (SAD / `sad.md`, **every** ADR under `adr/`, and tech indexes `ARCHITECTURE-OPEN-DECISIONS`, `IMPLEMENTATION-READINESS`):** each carries a short `.uk.md` **summary** (the decision + why, not a full translation). This is per-ADR — a new ADR means a new `<adr>.uk.md` in the same commit. The English file is the only authority; the `.uk` summary is a reading aid. Product docs (idea-brief, PRD, CONTEXT, BACKLOG, DECISION-LOG) keep full `.uk.md` copies as before.
- **Exception — implementation artifacts (`data-model.md`, `contracts/openapi.yaml`, `tasks/`, `test-plan.md`):** these are code-adjacent and stay **English-only**. Instead of per-file `.uk` siblings, each feature has **one** `overview.uk.md` — a compact Ukrainian reading aid covering data-model + API + tasks + tests together. The English files are the only authority; never derive anything from `overview.uk.md`.
- **Exception — design artifacts.** Per-feature `design-spec.md` + `design-sync-report.md` are design-adjacent and stay **English-only** (covered by the feature's `overview.uk.md`). Cross-cutting design docs under `docs/design-system/` follow the deep-technical rule: substantive ones (`NAVIGATION-MAP.md`, `UX-REVIEW.md`) carry a short `.uk.md` **summary**; pure indexes/data (`README.md`, `color-tokens.json`, `brand/`) are English-only. The top-level `TRACEABILITY-AUDIT.md` keeps a short `.uk.md` summary like the other tech indexes.
- Never derive requirements, decisions, or RICE/feasibility values from a `.uk.md` file. If the English and Ukrainian versions disagree, **the English `.md` wins** — fix the `.uk.md`.
- The root file `ideas.md` is the original founder note (mixed Ukrainian/English) and is exempt — historical input, not maintained documentation.
- Verbatim user quotes inside a doc (e.g. the "Raw idea" section) may be paraphrased into English in the `.md`; the original-language wording is preserved in the `.uk.md`.

## Markdown style — Obsidian (IMPORTANT)

**All docs in `docs/` are read through Obsidian**, so author them in Obsidian-flavored Markdown, not plain GitHub Markdown:

- **Internal links = wikilinks `[[...]]`, not `[text](path)`.** Because every brief shares the basename `idea-brief.md`, link with a **path-qualified wikilink + alias**: `[[features/authentication/idea-brief|Authentication brief]]`. For the glossary, the basename is unique, so `[[CONTEXT]]` (and `[[CONTEXT.uk]]` for the Ukrainian copy) is fine. English docs link only to English docs; `.uk.md` docs link only to `.uk.md` docs.
- **Tags** — two intentional uses, nothing else:
  - **Classification tags** go in frontmatter `tags:` (e.g. `tags: [sdlc/stage-01, feature/authentication, mvp, health]`), hierarchical `tag/subtag` form. These describe *what the doc is*.
  - **Workflow tags** are inline `#tag` on individual checklist items in [[BACKLOG]] only (`#blocker`, `#legal`, `#post-mvp`, `#adr-later`, …). These are *functional* — the Tasks query blocks filter on them (`tags include #blocker`), so they earn their place. Don't scatter inline `#tag` in ordinary prose anywhere else — it pollutes the tag pane.
  - Reuse the same tag words across docs so Obsidian search/graph stays coherent.
- **Frontmatter (YAML) is metadata Obsidian indexes.** Two stable shapes — don't invent per-doc keys:
  - **English source docs**: full process frontmatter — `status`, `owner`, `reviewers`, `updated_at`, `feature_size`, `stage`, `ticket`, `value_score`, `feasibility_state`, `tags`. Pending values stay empty (`ticket: ""`), never `<placeholder>` — placeholders read as broken in Obsidian's Properties panel.
  - **Ukrainian `.uk.md` copies**: a lighter reading subset — `status`, `owner`, `updated_at`, `value_score`, `feasibility_state`, `tags`, plus the two that mark them as copies: `lang: uk` and `source_of_truth: <english-file>`.
  - Add a new field to the whole family at once (all EN, or all UK), not one-off, so Dataview-style queries stay reliable.
- Prefer relative wikilinks that resolve inside the vault (the vault root is the repo root or `docs/` — keep links working from `docs/`). Don't hand-write `../../` relative paths for internal links; let Obsidian resolve the wikilink.
- **Mermaid diagrams** (Obsidian renders them natively) — reach for one when a picture genuinely beats prose: a flow, a state machine, an entity relationship, and **especially a sequence of interactions between actors/systems** (sequence diagrams are the preferred default — auth handshakes, upload→parse→validate→save flows, consent exchanges read far clearer as a diagram than as paragraphs). Rules of thumb:
  - Use a diagram to **replace** bulky text, not to decorate it. If the prose is already short and clear, no diagram.
  - One diagram should carry one idea. Don't cram a whole feature into a single sprawling graph.
  - Keep node/actor labels in the project's domain vocabulary (see [[CONTEXT]]) so the diagram and the glossary agree.
  - Briefs (stage 01) stay mostly prose; diagrams earn their place more in PRD/architecture/ADR where flows get concrete. Add them where they cut reading effort, not by default.

  ````
  ```mermaid
  sequenceDiagram
    actor U as User
    participant App
    participant AI as Parsing service
    U->>App: Upload lab result (PDF/image)
    App->>AI: Extract biomarkers
    AI-->>App: Draft values
    App->>U: Show for validation
    U->>App: Confirm / correct
    App->>App: Save (original file + values)
  ```
  ````

## SDLC workflow

This project follows the `sdlc` plugin's staged pipeline. **Stages 01→03.5 are complete** for all 7 MVP features (ideation, PRDs, architecture + ADRs, data-models, OpenAPI contracts, task breakdowns, test-plans, design-specs, and the Penpot screen designs), and **implementation of the MVP is done** — every per-feature tracker task is `done` except two intentionally-deferred items (F4 T6 catalog cache, F3 T0 PoC accuracy spike), each documented as such in its tracker. (F4 T2 LOINC seed loader was completed 2026-07-24 as the admin "Download latest pack" feature — download API + pack loader + linguistic-variant multi-language reference, delivered with F0 T7.) Stage-01 ideation artifacts:

- `docs/CONTEXT.md` — domain glossary (terms, invariants, out-of-scope). Read this first to learn the domain vocabulary (Analysis, Biomarker, lab vs clinical reference range, cross-biomarker rule, booking window, etc.).
- `docs/features/<slug>/idea-brief.md` — one 15-section idea brief per MVP feature, all `status: Confirmed`.
- `docs/DECISION-LOG.md` — rationale log (ADR-style): the *why* behind every product/tech decision (D1–D16) and the alternatives weighed. All decided; each brief's §15 links back here. Read it before PRD/architecture so you don't re-litigate settled calls.

MVP features and their slugs (RICE priority, higher = built first):

| Slug | Feature | RICE | Approach |
|---|---|---|---|
| `authentication` | Social login (Google/Apple/Meta) | 50 | A |
| `localization` | i18n EN/DE/UA + canonical coded catalog | 33 | A |
| `onboarding-profile` | Baseline questionnaire + profile | 27 | A |
| `checkup-scheduling` | Templates + per-country wait-time windows | 25 | C |
| `lab-result-parsing` | AI extract → validate → color-code + summary | 25 | C |
| `biomarker-monitoring` | Dual lab/clinical norms + cross-flags | 9 | C |

Backlog features F7–F12 (doctor-visit prep, guardian accounts, FHIR integrations, voice/AI assistant, auto-booking, family calendar), plus cross-cutting tracks (monetization, legal/compliance, market, tech direction), are tracked in the living product index [[BACKLOG]]. That file is the **Mission Control** — start each session there; it links out to briefs rather than duplicating them.

The MVP backend + mobile app are built (see **Working notes** for build/run/test commands). Remaining work is the two deferred tracker items above and the "prod swap" integrations (real VLM extractor for F3, Hangfire recurring triggers, FCM push) — all already isolated behind ports, so swapping a dev adapter for the real service is a local change. All upstream docs (PRD → architecture → data-model → contracts → tasks → test-plan → design-spec) exist per feature; build from the English sources, never the `.uk.md`. See [[READING-GUIDE]] for how the artifacts chain together.

## Domain constraints that shape every decision

These are product invariants (full list in `docs/CONTEXT.md`) — treat them as non-negotiable when writing any downstream doc or code:

- **The app informs; it never diagnoses.** Any AI summary, color status, or cross-biomarker flag is informational only and always points the user to a professional. Crossing this line risks EU MDR / AI-Act medical-device classification — the central regulatory risk. This is settled as the strictly-informational framing ([[DECISION-LOG]] D1); a lawyer's sign-off on the intended-purpose wording is the one remaining step before F2/F3 build.
- **AI-extracted values always pass explicit user validation before being saved.**
- **Reference ranges always carry their layer (lab vs clinical) and locale (country/sex/age).** A bare number is never stored as a norm.
- **Localization is data, not schema** — adding a language or country must never require a schema change.
- Medical data is special-category personal data (GDPR Art. 9); a DPIA is effectively mandatory and data minimization applies from the first field.

## Working notes

- Workflow/Agent subagents work on this deployment (verified 2026-07-16 executing the step-0 scaffold via subagent-driven development). They inherit the session model; no override needed.
- **Stack scaffolded** (step-0 complete — see [[superpowers/specs/2026-07-16-step-0-scaffold-design]] and [[superpowers/plans/2026-07-16-step-0-scaffold]]). The .NET server-side solution (API + admin + modules) lives in **`server/`** at the repo root; `docs/` stays at the root. Solution is a .NET 10 modular monolith, `Sentra.slnx`, Central Package Management. Commands (run from **`server/`**, bash):
  - Build: `dotnet build Sentra.slnx`
  - Unit tests (no Docker): `dotnet test tests/Sentra.UnitTests`
  - Integration tests (needs Docker): `dotnet test tests/Sentra.IntegrationTests`
  - E2E tests (browser-driven admin Blazor UI; needs Docker + a one-time Playwright browser install — `powershell -ExecutionPolicy Bypass -File tests/Sentra.E2ETests/bin/Debug/net10.0/playwright.ps1 install chromium`): `dotnet test tests/Sentra.E2ETests`
  - Local Postgres: `docker compose up -d` / `docker compose down`
  - **Run the whole admin backend (Aspire, recommended):** `dotnet run --project src/Aspire/Sentra.AppHost` — provisions Postgres + pgweb (needs Docker), applies migrations + seeds the bootstrap admin, launches `Sentra.AdminServer`, and opens the Aspire dashboard (traces/metrics/logs). One command for everything the admin backend needs.
  - Run API (standalone): `dotnet run --project src/Hosts/Sentra.Api`
  - Run admin (standalone, needs a Postgres on `localhost:5432`): `dotnet run --project src/Hosts/Sentra.AdminServer`
  - Add a package (CPM — resolves & pins centrally in `Directory.Packages.props`; never hand-write `Version` in a csproj): `dotnet package add <name> --project <path>`
  - Add EF migration (once a feature defines a DbContext): `dotnet ef migrations add <Name> --context <DbContext> --project <Infrastructure proj> --startup-project src/Hosts/Sentra.Api`
- **Solution layout** (all under `server/`): Clean Architecture per module — `src/Modules/<M>/Sentra.<M>.{Domain,Application,Infrastructure,Ports}` (7 modules: Identity, Catalog, Profile, Scheduling, Parsing, Monitoring, Admin); hosts in `src/Hosts` (`Sentra.Api`, `Sentra.AdminServer`); shared code in `src/BuildingBlocks` (`Sentra.SharedKernel` pure, `Sentra.Persistence` EF); local-dev orchestration in `src/Aspire` (`Sentra.AppHost` = .NET Aspire orchestrator, `Sentra.ServiceDefaults` = shared OTel/health/resilience). The dependency rule (Domain → SharedKernel; Application → Domain; Infrastructure → Application + Persistence; Ports → Application; hosts → Infrastructure + Ports) is **enforced by project references** — a Domain→Infrastructure `using` fails to compile. The `snake_case` enum converter + UUIDv7 + `EntityBase` live in BuildingBlocks; feature modules reuse them. The repo is a polyglot monorepo: `server/` (this .NET solution), `mobile/` (Flutter, later), `k8s/` (deploy), `docs/` (product + design docs).
- **Enum storage convention (EF Core ⇄ API):** enums are stored as **lowercase `snake_case`** strings (e.g. `processing`, `deactivated`, `lab`, and multi-word `doctor_advice`, `not_provided`, `worth_watching`) to match the OpenAPI contracts and JSON payloads. The default `.HasConversion<string>()` stores the C# member name verbatim (PascalCase, e.g. `DoctorAdvice`) — instead use a converter with an explicit value map (or a global convention) so the C# `DoctorAdvice` ⇄ DB/JSON `doctor_advice`. This keeps DB column literals, partial-index `HasFilter` predicates, and the API contract all in agreement. The C# enum members stay PascalCase in code; only the stored/serialized form is lowercase snake_case. (The `.HasConversion<string>()` calls in the per-feature `data-model.md` EF sketches are shorthand for this mapped converter.)

---
> Source: [HeliosPersonal/sentra](https://github.com/HeliosPersonal/sentra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
