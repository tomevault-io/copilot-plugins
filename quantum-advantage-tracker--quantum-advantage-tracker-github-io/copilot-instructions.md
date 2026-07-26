## quantum-advantage-tracker-github-io

> npm run dev      # Next.js dev server on http://localhost:3000

## Commands

```bash
npm run dev      # Next.js dev server on http://localhost:3000
npm run build    # Static export → ./out (no `start` needed for the deployed site)
npm run lint     # ESLint (flat config extending eslint-config-next)
npm run format   # Prettier write across the repo
```

There is no test runner. CI (`.github/workflows/ci.yml`) runs `npm run lint` and `npm run build` on every PR — both must pass. Pre-commit runs `lint-staged` (eslint --fix + prettier) via `simple-git-hooks`. Node 24.x is pinned in `package.json#engines`.

The deployed build sets `PAGES_BASE_PATH` (see `next.config.ts` and `.github/workflows/deploy.yml`); to mirror the deployed paths locally, set the same env var before `npm run build`.

## Domain language

`CONTEXT.md` defines the project's terminology (Tracker, Submission, Instance, Category, Runtime to solution) and lists _forbidden synonyms_ for each. **Read it before naming anything user-facing or writing prose** — using "track", "entry", "problem", "status", "TTS", etc. is a documented mistake. Categories are not freeform strings; they're the three values in `src/lib/categories.ts` (`Active candidates`, `Superseded candidates`, `Baseline benchmarks`) and drive the primary tab filter on every tracker page.

## Architecture

**Next.js 16 App Router, statically exported** (`output: 'export'` in `next.config.ts`) and deployed to GitHub Pages. There is no server runtime — every page must work as static HTML + client JS.

**Three parallel trackers, one shared shell.** OE / VP / CVP each have:

- A route at `src/app/trackers/<tracker>/page.tsx` that imports its `data/<tracker>/submissions.json` and the relevant instance file (`circuit-models.json` for OE/CVP, `hamiltonians.json` for VP) via JSON import assertions, then renders a tracker-specific `SubmissionsTable.tsx` next to it.
- A _thin_ tracker-specific `SubmissionsTable.tsx` (in the same route folder) that adapts the tracker's submission shape to the **generic shared** `<SubmissionsTable>` in `src/components/SubmissionsTable.tsx` by passing `getInstanceId`, `getInstanceUrl`, `getQubits`, `getGates`, a `valueColumn` renderer, and a `chart` config.

When adding a feature that should appear in all three trackers, change the shared component in `src/components/` and pass new props through each route's adapter — don't duplicate logic across the three `app/trackers/<tracker>/SubmissionsTable.tsx` files.

The shared table uses `nuqs` to mirror filter state (`category`, `instance`) into URL query params; keep that contract intact when adding filters so deep links continue to work. The submission types in `src/types/submissions.ts` (`OESubmission`, `VPSubmission`, `CVPSubmission`) all extend `BaseSubmission` — put cross-tracker fields there.

## Data flow: how submissions get in

Submissions are **never edited by hand**. The pipeline is:

1. Contributor opens a GitHub issue using one of the templates in `.github/ISSUE_TEMPLATE/` (six templates: regular + "problem" variants for each of the three trackers).
2. A maintainer adds the `verified` label.
3. `.github/workflows/verify-issue.yml` fires: it picks the matching template based on the `path: <tracker>` and `problem submission` labels, parses the issue body with `issue-ops/parser`, runs `.github/workflows/parse-submission.js` to coerce numeric fields and prepend `createdAt` / `url`, appends the result to `data/<tracker>/submissions.json` via `jq`, and opens a PR.

If you change a submission field, update **all** of: the relevant issue template(s), `parse-submission.js` (numeric field list), the `*Submission` type in `src/types/submissions.ts`, and the table/chart adapters that read it. The exponential-number marker hack in `parse-submission.js` exists because `JSON.stringify` would otherwise quote scientific notation as a string — preserve that pattern if extending the numeric list.

## Conventions worth knowing

- Path alias `@/*` → `src/*` (use it; relative `../../../` is reserved for importing `data/`).
- Prettier config is inside `package.json` (`singleQuote`, `printWidth: 100`) plus three plugins: `organize-imports`, `packagejson`, `tailwindcss` — running `npm run format` will reorder imports and Tailwind classes.
- Tailwind v4 (no `tailwind.config`); design tokens live in `src/app/globals.css`. UI primitives in `src/components/ui/` follow the shadcn/Radix pattern (see `components.json`).
- Client components live where they're used; the shared `SubmissionsTable` is `'use client'` because of `nuqs` and `useState`. Tracker page entry points stay server components and pass JSON data down.

## Analytics (GDPR)

Google Analytics 4 is loaded under **Google Consent Mode v2** (`src/components/analytics/`, shared logic in `src/lib/consent.ts`). gtag boots with every consent category defaulted to `denied`; only `analytics_storage` upgrades to `granted` after the user clicks Accept in the `CookieConsent` banner, and the choice is persisted in `localStorage`. All of it is inert unless `NEXT_PUBLIC_GOOGLE_ANALYTICS_ID` (a GA4 `G-XXXXXXXX` id) is set at build time — provided in production via the `NEXT_PUBLIC_GOOGLE_ANALYTICS_ID` GitHub Actions repository variable (see `deploy.yml`). To test locally, set `NEXT_PUBLIC_GOOGLE_ANALYTICS_ID` in `.env.local` before `npm run dev`.

**Page views on in-app navigation.** The app navigates via `next/link` (soft navigation using the History API), so there is no full page reload between routes. Client-side navigations are counted by GA4's **Enhanced Measurement** ("Page changes based on browser history events", enabled on the property) — `page_view` fires "each time the page loads _or_ the browser history state is changed". There is deliberately **no** manual route-change tracking in the code; adding one would double-count every soft navigation. Keep the Enhanced Measurement option enabled on the GA4 property.

---
> Source: [quantum-advantage-tracker/quantum-advantage-tracker.github.io](https://github.com/quantum-advantage-tracker/quantum-advantage-tracker.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
