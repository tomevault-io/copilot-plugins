## squadridge-v1

> This repository contains **SquadRidge**, a pilot‑stage verified dialogue infrastructure for structured, facilitator‑led cross‑border dialogue.

# GitHub Copilot instructions for this repository

## What this repo is

This repository contains **SquadRidge**, a pilot‑stage verified dialogue infrastructure for structured, facilitator‑led cross‑border dialogue.

SquadRidge is **not** a generic chat app.
It focuses on:
- Verified access (pilot‑scoped identity / attribute proofs, Semaphore‑style).
- Pseudonymous small‑squad matching and structured session rooms.
- Facilitator and moderator affordances, including safety workflows.
- Encrypted‑at‑rest session chat and RLS‑enforced access.
- Public, anonymous, timestamped ledger‑style outcome records for squad consensus.

Impact claims are intentionally modest: this repo supports *serious bounded pilots*, not “solved global conflict prevention” or fully operator‑blind, public early‑warning infrastructure.

## Tech stack and architecture

- Frontend: React + Vite + TypeScript.
- Styling: Tailwind CSS.
- Backend: pure BaaS via Supabase (PostgreSQL + RLS, Auth, Realtime, Edge Functions).
- No Node server tier and **no local Redis** dependency.
- Edge rate limiting (when enabled) uses Upstash Redis via REST from `supabase/functions/rate-limit/`.

When older docs mention `docker-compose.yml` or a local Redis stub, treat them as obsolete; follow the current Supabase‑only architecture.

## Environment and configuration

When generating or modifying code that relies on environment variables, assume:

- `VITE_SUPABASE_URL` — Supabase project URL (`https://<ref>.supabase.co`).
- `VITE_SUPABASE_PUBLISHABLE_KEY` (preferred) or legacy `VITE_SUPABASE_ANON_KEY`.
- `VITE_SITE_URL` — deployed origin (used for magic links).
- Optional flags such as `VITE_ENABLE_DEMO_SQUAD` and any other documented `VITE_*` keys in `.env.example`.

Guidelines for env usage:

- Never hard‑code secrets, keys, or project refs.
- Use `import.meta.env.VITE_*` on the client.
- Assume `.env.example` documents the intended shape; keep new envs documented there.
- `.env*` files must **never** be committed; they are gitignored.

## How to run, build, and test

When adding instructions or scripts, align with the existing script surface:

- `npm run dev` — Vite dev server for local development.
- `npm run build` — typecheck + production build, also writes `dist/partner-one-pager.html`.
- `npm run build:e2e` — build for Playwright / demo smoke using `.env.e2e`.
- `npm run preview` — preview production build.
- `npm run e2e` — hermetic Playwright suite (default project).
- `npm run e2e:staging` — staging‑only specs (`*-staging.spec.ts`).
- `npm run demo:capture` — scripted tour screenshots under `test-results/demo-capture/`.
- `npm run gen:partner-one-pager` — regenerate the printable partner one‑pager from `docs/business/pilot-partner-one-pager.md`.

When adding new tests, prefer Playwright for E2E flows that touch auth/RLS/Realtime and align with the existing `e2e` and `e2e:staging` project patterns.

## Supabase, migrations, and CI

This repo assumes a Supabase project is linked and migrations are applied via CI:

- Supabase resources (tables, RLS, auth, Realtime, Edge Functions) live under `supabase/`.
- Migrations are the source of truth for schema.
- After linking Supabase + GitHub, the `Deploy Supabase to production` workflow on `main` should succeed.

The Supabase deploy workflow (`.github/workflows/deploy-supabase-production.yml`):

- Uses repository secrets:
    - `SUPABASE_ACCESS_TOKEN` — classic PAT (`sbp_*`), *not* anon or service_role.
    - `SUPABASE_DB_PASSWORD` — database password for migrations.
    - `SUPABASE_PROJECT_ID` — 20‑character project ref (e.g. the `<ref>` in `https://<ref>.supabase.co`).
- Validates token and ref shape before running.
- Deploys database migrations only; static frontend is shipped via a separate pipeline.

When Copilot edits CI YAML:

- Preserve existing validations and comments, especially around PAT formats.
- Do not introduce new secrets without also updating documentation.
- Keep deploy steps idempotent and safe to re‑run.

## Frontend hosting

The static app is deployed separately from Supabase migrations:

- Use the same `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY` (or anon), and `VITE_SITE_URL` as in local production builds.
- Set env vars in the hosting provider (Vercel, Netlify, Cloudflare Pages, etc.), not just in local `.env`.
- `.github/workflows/deploy-frontend.yml`:
    - Runs tests and `npm run build` (with `VITE_ZK_STUB=false`).
    - Uploads `dist/` as a build artifact for download / attachment to a host if desired.

When Copilot touches build or deploy logic, keep these invariants:

- Builds must respect feature flags like `VITE_ZK_STUB` and `VITE_ENABLE_DEMO_SQUAD`.
- The default pipeline assumes real ZK / security‑relevant code is *on* in production; stubs are for demos and development only.
- Do not bypass tests or readiness checks without an explicit, documented reason.

## Domain concepts and flows

Key flows the codebase supports today:

1. **Verify**
    - Pilot‑scoped access and attribute verification (Semaphore‑style).
    - Treat ZK‑ish verification as a first‑class concern; keep flows composable for future cryptographic upgrades.

2. **Convene**
    - Small squads enter a structured session room with facilitator + moderator affordances.
    - Pseudonymous identities are the norm; avoid leaking real‑world identifiers into UI or logs.

3. **Record outcomes**
    - Consensus records can be published to an anonymous, timestamped ledger surface.
    - Ledger records are intentionally lean: no personal identifiers, focus on outcome statements and minimal metadata.

Roadmap / non‑shipped work:

- CSI / early‑signal surfaces and any operator‑blind public early‑warning infrastructure are explicitly **roadmap** and may be scaffolded only, unless `CURRENT_STATUS.md` says otherwise.
- When in doubt, check `CURRENT_STATUS.md` and relevant docs under `docs/`.

## Demos and investor tooling

The repository includes structured demo surfaces:

- `/investors` — public investor mini‑page (sanitized, link‑safe, no gated decks or sensitive scenarios).
- `/pitch-deck-hub` — moderator‑only interactive pitch hub (uses content also available as static HTML under `public/pitch-deck-hub/`).
- `/demo` → `/admin/demo-hub` — moderator‑only presenter hub for scenario selection and tour control.
- `?notes=1` — presenter notes overlay on tour URLs (desktop only).
- `/session/demo-session-001` — static offline demo session page with seeded messages and scripted interventions.

When Copilot changes demo behavior:

- Do not accidentally turn demo‑only flows into production flows.
- Keep “offline” demo behavior clearly separated from live Realtime / RLS‑backed sessions.
- Preserve presenter notes, scenario wiring, and URLs that are referenced by docs or slides.

## Documentation to rely on

When you need additional context, prefer these files before guessing:

- `docs/README.md` — curated documentation index (product, technical, security, ops, ADRs).
- `CURRENT_STATUS.md` — current scope and what is actually live vs. roadmap.
- `DILIGENCE_OVERVIEW.md` — diligence‑oriented summary.
- `SECURITY.md` and `docs/security/threat-model.md` — security posture, boundaries, and non‑goals.
- `docs/operations/pilot-runbook.md` and `docs/operations/incidents.md` — pilot operations and incident handling.
- `docs/product/metrics-spec.md` — metrics definitions.
- `docs/business/*` and `docs/pitch/*` — partner, fundraising, and pitch material.

If a change would contradict these docs, prefer updating docs and code together.

## Coding style and guidelines

When generating or editing code, Copilot should:

- Prefer TypeScript with strict, explicit typings.
- Use modern React (function components, hooks), no legacy class components.
- Use Tailwind utility classes for styling; avoid ad‑hoc inline styles except for tiny one‑off cases.
- Respect existing component boundaries, file structure, and naming conventions.
- Maintain accessibility (semantic HTML, ARIA where appropriate, keyboard navigation).
- Keep user‑facing copy sober and safety‑aware; no sensationalism about conflict, trauma, or violence.

Security and safety:

- Treat anything that touches auth, RLS, encryption, Edge Functions, ZK primitives, or moderation pipelines as **security‑touching**.
- Security‑touching work must be small, explicit, and testable; never slip in unnoticed behavior changes.
- Never log secrets, tokens, or personally identifying information.
- Prefer “least privilege” when interacting with Supabase, feature flags, or internal APIs.

## Contributing and PR flow

For any non‑trivial change, expect to go through the documented contribution flow:

- `CONTRIBUTING.md` describes branching, checks, and PR expectations.
- `.github/pull_request_template.md` includes a **Security‑touching change?** section.
- `docs/operations/branch-protection.md` documents the intended branch protection setup.

When Copilot prepares PR descriptions or checklists:

- Call out any security‑touching areas (RLS, migrations, Edge Functions, ZK, encryption, auth).
- Reference relevant docs (e.g., threat model, pilot runbook) rather than re‑explaining them.
- Make it easy for reviewers to see what is pilot‑only vs. intended for future generalization.

## What not to do

- Do **not** introduce a Node.js backend, GraphQL layer, or local Redis unless the docs explicitly change direction.
- Do **not** weaken or bypass Supabase RLS for convenience in tests or demos.
- Do **not** auto‑invent new cryptographic or ZK schemes; follow existing patterns and stubs.
- Do **not** bloat the public ledger / outcome records with PII or unnecessary data.
- Do **not** expand scope from “bounded pilots” to “global early‑warning infrastructure” without matching docs and safeguards.

Keep changes aligned with a pilot‑stage, safety‑first, verifiable but privacy‑respecting dialogue infrastructure.

---
> Source: [brass4698-coder/squadridge-v1](https://github.com/brass4698-coder/squadridge-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
