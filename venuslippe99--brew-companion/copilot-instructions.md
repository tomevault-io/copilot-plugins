## brew-companion

> This repo is a Vite + React + TypeScript kombucha companion app with Supabase-backed auth, batch tracking, fermentation guidance, F2 planning, guides, and beginner-friendly onboarding.

# AGENTS.md

## Purpose

This repo is a Vite + React + TypeScript kombucha companion app with Supabase-backed auth, batch tracking, fermentation guidance, F2 planning, guides, and beginner-friendly onboarding.

The priority is clear, safe guidance for non-expert brewers while keeping batch lifecycle data consistent across UI, derived timing, and persisted records.

## Repo Shape

- `src/pages`: route-level screens such as onboarding, login, batch creation, batch list, and batch detail.
- `src/components`: UI building blocks, layout, auth guards, shared status components, and the F2 setup wizard.
- `src/lib`: batch domain types, stage labels, next actions, F1/F2 timing logic, F2 planning, and F2 persistence helpers.
- `src/contexts`: auth and user preference state; `UserContext` also mirrors some settings to `localStorage`.
- `src/content`: static guide content.
- `src/integrations/supabase`: client setup and generated database types.
- `supabase/migrations`: schema history for batches, reminders, timeline data, F2 setup, recipes, bottles, guides, and assistant data.

## Core Product Rules

1. Prefer clarity over cleverness.
2. Keep guided flows explicit, especially in `src/pages/NewBatch.tsx`, `src/pages/BatchDetail.tsx`, and `src/components/f2/F2SetupWizard.tsx`.
3. Do not present fermentation timing or carbonation advice as certainty; this app uses estimates.
4. Preserve existing batch history, timeline events, reminders, bottle plans, and recipe traceability unless the task explicitly requires a migration.
5. Do not silently change stage progression, timing thresholds, carbonation risk logic, or next-action text without tracing all consumers.
6. When brewing rules are unclear, inspect the current helpers and schema before adding new assumptions.

## Working Style

1. Inspect relevant route files, domain helpers, and Supabase usage before editing.
2. Reuse existing types from `src/lib/*.ts` and `src/integrations/supabase/types.ts` instead of redefining domain shapes.
3. Keep business logic in `src/lib` when practical; keep pages and components focused on presentation and orchestration.
4. Keep changes tight to the requested task. Do not refactor unrelated UI or schema areas casually.
5. If work touches more than 3 files, stage transitions, persistence, or multi-step flows, write a short plan before editing.
6. If work touches persisted data, identify both the frontend write path and the affected Supabase tables first.

## Actual Data Flow To Respect

1. Auth and user state flow through `AuthProvider` and `UserProvider`; `UserContext` always mirrors preferences to `localStorage` and syncs supported preference fields to `profiles` when authenticated.
2. New batches are created from `src/pages/NewBatch.tsx` and inserted into `kombucha_batches`.
3. Batch detail reads from `kombucha_batches`, `batch_reminders`, `batch_stage_events`, and `batch_logs`, then derives timing UI with `src/lib/batch-timing.ts`.
4. F2 setup starts in `src/components/f2/F2SetupWizard.tsx`, planning lives in `src/lib/f2-planner.ts`, and persistence lives in `src/lib/f2-persistence.ts`.
5. Starting F2 does not only update `kombucha_batches`; it also creates or updates `batch_f2_setups`, `batch_f2_bottle_groups`, `batch_bottles`, `batch_bottle_ingredients`, `batch_stage_events`, and `batch_logs`.
6. Shared stage labels and default next actions live in `src/lib/batches.ts`; do not fork those rules in components.

## Brewing And Lifecycle Rules

1. The current batch stage model is:
   `f1_active`, `f1_check_window`, `f1_extended`, `f2_setup`, `f2_active`, `refrigerate_now`, `chilled_ready`, `completed`, `archived`, `discarded`.
2. If you add or change a stage, update all dependent UI and persistence paths, including stage badges, progress displays, next actions, timeline text, F2 actions, and any Supabase enum usage.
3. F1 timing estimates are derived in `src/lib/batch-timing.ts` from brew date, room temperature, target preference, and starter ratio. Keep the explanation text aligned with the actual calculation.
4. F2 planning depends on bottle size, headspace, flavour additions, carbonation target, and ambient temperature. Changes here can affect both safety messaging and persisted bottle plans.
5. Recipe traceability matters. Preset recipes, user recipes, recipe snapshots, bottle groups, and bottle ingredient rows should stay compatible with existing saved setups.
6. When changing a batch stage from the UI, preserve timeline/history context by checking whether `batch_stage_events` and `batch_logs` also need updates.

## UX Rules

1. This app is aimed at beginners first.
2. Prefer step-by-step flows and clear review states over dense expert forms.
3. Use calm, practical language. Good copy explains what the user should do next.
4. Surface current stage, current risk, timing window, and next action clearly.
5. Do not remove helpful context from overview cards, timeline entries, reminders, or F2 review summaries without a good reason.

## Data Safety Rules

1. Avoid destructive schema or data changes unless explicitly requested.
2. Prefer additive migrations in `supabase/migrations` over breaking rewrites.
3. Keep `src/integrations/supabase/types.ts` aligned with schema changes.
4. Be careful with backwards compatibility for saved batches, saved F2 setups, recipe snapshots, and profile preferences.

## Validation

After code changes, run the commands this repo supports:

1. Typecheck: `npx tsc -b`
2. Lint: `npm run lint`
3. Tests: `npm run test`
4. If the change affects the production bundle or routing/build behavior, also run: `npm run build`

Then summarize:

1. Which files changed
2. What was implemented
3. Any follow-up risks or recommended next steps

## Copy Architecture Rules

1. Keep meaningful user-facing copy in feature-specific copy modules under `src/copy/` when practical.
2. Prefer feature-level files such as `src/copy/f1-new-batch.ts`, `src/copy/f2-setup.ts`, or `src/copy/home.ts` over one global copy file.
3. Extract and wire page titles, section headings, helper text, field labels, placeholders, warnings, empty states, CTA labels, review copy, and user-facing toast messages when working in a feature.
4. Keep dynamic user-facing strings in typed copy helpers/functions instead of assembling them inline in components.
5. Do not rewrite existing copy during extraction unless the task explicitly asks for copy changes.
6. Very small local fragments such as tiny units or punctuation glue may remain inline when extraction would reduce clarity.
7. New multi-step flows and major page work should follow the copy-module structure by default.

## Response Expectations

1. Start by stating which files or areas you will inspect.
2. For non-trivial work, share a short plan before editing.
3. After implementation, report concrete outcomes.
4. Flag uncertainty explicitly instead of guessing, especially around brewing guidance or schema intent.

## Priorities For This Repo

Prioritize correctness in this order:

1. Data integrity
2. Batch lifecycle consistency
3. User clarity
4. Maintainability
5. Visual polish

## Execution Plans

Use an execution plan as described in `PLANS.md` for any non-trivial task, especially work involving:
1. More than 3 files
2. Batch lifecycle or stage logic
3. Supabase persistence or migrations
4. F2 setup, bottle planning, or recipe persistence
5. Multi-step features or refactors

When a plan is needed:
1. Create or update a dedicated markdown plan file. Use `plans/<date>-<name>.md` if the `plans/` directory exists; otherwise create that directory first.
2. Follow `PLANS.md` exactly
3. Keep the plan current as work progresses
4. Implement milestone by milestone
5. Run validation after each milestone before moving on
6. Do not ask for "next steps" between milestones unless a true product decision is missing

## Things To Avoid

1. Do not introduce broad redesigns unless requested.
2. Do not rename core brewing concepts or stage names casually.
3. Do not invent new fermentation or carbonation rules when the repo already has helpers for them.
4. Do not change persistence behavior without checking the write path, read path, and generated types.
5. Do not remove timeline, reminder, next-action, or bottle-plan context that the current flows rely on.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Venuslippe99)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Venuslippe99)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
