## deezer-explorer

> Deezer Explorer is a static, responsive website that lets a user:

# AGENTS.md — Operating Guide for Deezer Explorer

## Project Overview

Deezer Explorer is a static, responsive website that lets a user:

1. Search for an artist
2. Select one result
3. Browse that artist's albums as cards with cover art
4. Open an album and view tracklist, cover, and release date

Data source is Deezer public API only. The project target is GitHub Pages, with no custom backend, no authentication, and no database for v1.

## v1 Objectives and Current Scope

The current v1 scope is the single end-to-end flow defined in `PLAN.md` and summarized in `README.md`.

Mandatory Deezer endpoints for v1:

- `GET https://api.deezer.com/search/artist?q=...`
- `GET https://api.deezer.com/artist/{id}/albums`
- `GET https://api.deezer.com/album/{id}`

Constraints that must be preserved:

- Mobile-first, responsive UI
- Keyboard-accessible main flow
- Few dependencies
- Static deployment to GitHub Pages
- First page of API results only (no pagination in v1)

Out-of-scope items (do not add without explicit alignment): auth, user accounts, favorites, audio playback, backend/database, i18n, PWA, and extra search modes.

## Source of Truth Files

Use these files in this order before proposing or implementing any change:

1. `PLAN.md` — primary scope, stack direction, phases, risks, and decision points
2. `README.md` — repository-facing summary of current status and operational expectations

If there is any mismatch, treat `PLAN.md` as authoritative and propose a synchronized update to `README.md`.

## How Agents Must Use `PLAN.md` and `README.md`

Before proposing changes:

1. Read `PLAN.md` sections for scope, stack, phase boundaries, and known risks.
2. Read `README.md` to confirm the project's communicated status.
3. Explicitly identify which phase the proposed change belongs to.
4. Avoid jumping ahead to later phases unless requested.

Before implementing changes:

1. Confirm the change is inside current v1 scope.
2. Confirm it does not introduce stack drift (for now: Vite + TypeScript + DOM/fetch + plain CSS).
3. Confirm it does not silently expand scope (for example pagination, playback, auth).

After implementing or changing decisions:

1. Update `PLAN.md` when scope, sequence, risk handling, or architecture decisions change.
2. Update `README.md` when repository status, setup instructions, or delivery expectations change.
3. Keep both files consistent in the same change set whenever practical.

## Commands and Operational State

Current repository state is early-stage and planning-first.

- `package.json` is not present yet.
- Standard scripts (`npm run dev`, `npm run build`, `npm run preview`) are planned in Phase 0 but may not exist until bootstrap is implemented.

Agent rule:

- Do not claim commands are available unless they are present in `package.json`.
- If commands are missing, state that clearly and either (a) implement the bootstrap when requested or (b) provide the planned command contract from `PLAN.md`.

## Quality and Validation Standards

For each delivered increment, validate at the level appropriate to the current phase:

- Functional: the exact step in the user flow works as defined.
- Error handling: loading, empty, and failure states are visible and specific.
- Accessibility baseline: keyboard operability and visible focus for primary interactions.
- Responsiveness: no broken layout at small mobile viewport and common desktop widths.
- Deployment readiness: verify assumptions for static output and GitHub Pages `base` behavior.

When applicable, include manual validation notes matching the phase criteria in `PLAN.md`.

## Rules for Small, Phase-Aligned Changes

Always propose and implement work in small increments tied to one phase at a time:

1. State the target phase.
2. Define the smallest testable deliverable.
3. Implement only what is needed for that deliverable.
4. Validate and report outcomes.
5. Document follow-up decisions or open risks.

Do not combine unrelated phase work in one change unless explicitly requested.

## What Must Not Be Invented, Changed, or Assumed

Without explicit alignment, do not:

- Invent features outside current v1 scope.
- Introduce a UI framework or major dependency shift.
- Assume Deezer browser access works across origins without validating CORS behavior.
- Add backend/database requirements as default architecture.
- Present the project as implemented when it is still in planning or partial phases.
- Change scope boundaries in docs silently.

When uncertainty exists, capture it as an open decision and request alignment before proceeding.

---
> Source: [Clever021/deezer-explorer](https://github.com/Clever021/deezer-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
