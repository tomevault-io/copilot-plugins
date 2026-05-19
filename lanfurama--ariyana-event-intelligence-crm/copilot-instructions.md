## ariyana-event-intelligence-crm

> > Loaded automatically by Claude Code at session start. Keep tight (~200 lines). For full architecture details see README.md; for refactor specs see `docs/superpowers/`.

# Claude Project Context — Ariyana Event Intelligence CRM

> Loaded automatically by Claude Code at session start. Keep tight (~200 lines). For full architecture details see README.md; for refactor specs see `docs/superpowers/`.

## Project

Event-intelligence CRM with React 19 + TS strict frontend (Vite), Express + Postgres backend (`api/`), and AI integrations (Gemini + Vertex + OpenAI). Solo dev. Active codebase optimization (sub-projects #1–#10) — see roadmap below.

## Conventions

- **Solo dev, commits straight to `main`.** No PRs, no review.
- **No CI yet (sub-project #7).** Pre-push hook runs `npm run typecheck && npm run typecheck:api && npm test` — that's the only gate. Never bypass with `--no-verify` unless explicitly authorized.
- **Pre-commit hook** runs `lint-staged` (ESLint --fix + Prettier on staged files). Blocks on lint errors.
- **No React Testing Library yet.** Component tests are deferred until a future "frontend test infra" sub-project. For UI changes, manual browser smoke is the only safety net — run after every commit, do not batch.
- **Pure helpers + custom hooks** are the testable layers in the frontend. Tests live alongside source: `*.test.ts`.
- **TS strict** is on except for `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` (deferred to sub-project #5 — see `STRICT_DEBT.md`).
- Daily scripts: `npm run lint | lint:fix | format | format:check | typecheck | typecheck:api | build | test | test:watch | test:coverage`.

## Where things live

- `STRICT_DEBT.md` — canonical tracker for `@ts-nocheck` / `@ts-expect-error` markers + deferred TS flags. Update its "Resolved" section when clearing markers.
- `docs/superpowers/specs/YYYY-MM-DD-<name>-design.md` — design specs (immutable once written).
- `docs/superpowers/plans/YYYY-MM-DD-<name>.md` — step-by-step implementation plans.
- `api/src/services/ai/` — AI provider abstraction (prompts in `prompts/`, scaffolded providers in `providers/`).
- `views/IntelligentDataView/EventModal/` — exemplar of refactor pattern (sub-project #4c done).

## Refactor roadmap status (as of 2026-05-07)

| Sub-project                                           | Status         | Notes                                                                                   |
| ----------------------------------------------------- | -------------- | --------------------------------------------------------------------------------------- |
| #1 Cleanup + tooling baseline                         | ✅ done        | Prettier, ESLint flat, TS strict, Zod env, husky                                        |
| #2 Test infra                                         | ✅ done        | Vitest, 183 tests across 20 files                                                       |
| #3 Structured logger / observability                  | pending        |                                                                                         |
| #4b Refactor `components/LeadDetail.tsx`              | 🔄 in progress | 1130 LOC `@ts-nocheck` (was 1998). Plan in 10 commits / ~5 sessions. Tasks 1–5/10 done. |
| #4c Refactor `views/.../EventModal.tsx`               | ✅ done        | Template for #4b. 7 sub-components + tested pure data fn.                               |
| #4d Refactor `views/LeadsView.tsx` (1914 LOC)         | pending        | Same playbook as #4b.                                                                   |
| #4 Refactor `api/src/routes/excelImport.ts` (973 LOC) | pending        | God file with 2 markers.                                                                |
| #5 API layer-ization + type normalization             | pending        | 5 markers remain (imap×2, excelImport×2 — also #4, managerReport).                      |
| #6 node-cron v4 migration                             | pending        | 1 marker (`scheduledReportsJob.ts`).                                                    |
| #7 GitHub Actions CI                                  | pending        | Replaces pre-push hook.                                                                 |
| #9 Bundle splitting / lazy loading                    | pending        |                                                                                         |
| #10 Documentation polish                              | pending        |                                                                                         |
| AI provider abstraction (out-of-band)                 | ✅ done        | Prompts extracted to `services/ai/prompts/`. Providers scaffolded.                      |

## Currently active: #4b LeadDetail refactor

- **Spec:** `docs/superpowers/specs/2026-05-07-lead-detail-refactor-design.md`
- **Plan:** `docs/superpowers/plans/2026-05-07-lead-detail-refactor.md` (10 tasks)
- **Approach:** Tab + custom hook decomposition (different from EventModal's pure-derive pattern because LeadDetail is genuinely stateful per-tab — 21 useStates split across 3 tab concerns).
- **Target structure:** `LeadDetail.tsx` <100 LOC orchestrator + `LeadDetail/{leadDetailHelpers, useLeadEdit, useLeadEnrichment, useLeadEmail, LeadInfoTab, LeadEnrichTab, LeadEmailTab}`.

### Done

| Task   | Commit    | Notes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ------ | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Task 1 | `188bcfb` | Extract pure helpers + 27 tests. `LeadDetail.tsx` itself untouched.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Task 2 | `311e164` | Swap inline helpers → imports. Removed inline `extractDomain`/`verifyEmail`/`parseResearchResult` (~310 LOC) + 16-call placeholder loop. -394 +9. Smoke pass on Info/Enrich/Email.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Task 3 | `aefd434` | Extract `useLeadEdit` hook. Owns `isEditing`/`editedLead` + prop-sync effect + handlers. Container destructures incl. `setEditedLead` (4 enrich-flow call sites still write to it). Smoke pass on Info edit flow.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Task 4 | `594f7e6` | Extract `useLeadEnrichment` hook. Owns 7 enrich `useState`s, rate-limit countdown effect, enrich-fields prop-sync effect, and 4 handlers (`handleEnrich` ~88 LOC + `handleApprove/RejectEmail` + `handleSaveEnrichment`). Signature is `(lead, editedLead, setEditedLead, onSave)` — `onSave` added to plan because handlers persist updates. -158 LOC. Smoke: error path verified end-to-end via Playwright (no live API key); success path proven by typecheck + verbatim copy.                                                                                                                                                                                                                                    |
| Task 5 | `95e92be` | Extract `useLeadEmail` hook. Owns 13 email `useState`s, email-tab data-load effect, email rate-limit countdown effect, and 7 handlers (incl. `loadEmailTemplates` ~73 LOC + `handleSendEmail` ~104 LOC). Signature `(lead, activeTab, setEditedLead, onSave)` — same +2 divergence as Task 4. Inline 16-pattern .replace() loop in `handleTemplateChange` swapped for `applyTemplatePlaceholders`. -323 LOC. Hook 382 LOC (under spec's 400-LOC split threshold). Smoke (Playwright): tab open → 7 templates loaded → 'Introduction' auto-selected → placeholders substituted → template switch updates subject/body → HTML/Preview toggle re-renders. AI draft / Send / Inbox / Upload not exercised (cost/safety). |

### Next session

JSX extractions (Tasks 6–8) — mechanical and low-risk now that all state is in hooks:

1. **Task 6: extract `LeadInfoTab`.** Pure presentational component. Receives `lead`, `editedLead`, `isEditing`, edit handlers as props.
2. **Task 7: extract `LeadEnrichTab`.** Receives the `useLeadEnrichment` return + `editedLead` for context.
3. **Task 8: extract `LeadEmailTab`.** Largest of the three sub-components — receives the `useLeadEmail` return.

Task 9 is the highest-risk: restore TS strict on `LeadDetail.tsx` (remove `@ts-nocheck`, fix all reported errors — by then the file should be a thin orchestrator). Task 10 closes with `STRICT_DEBT.md` update.

## Important decisions log

### LeadDetail refactor (2026-05-07)

- **Tab + hooks vs section-only split.** EventModal split JSX into sub-components and was done. LeadDetail has 21 `useState`s clustered by tab — splitting only JSX would leave all state in the orchestrator. Custom hooks encapsulate per-tab state cleanly. Spec §3.
- **`parseResearchResult` adds `fallbackKeyPerson` param.** The original closed over component state `enrichKeyPerson`. Pure version makes the dependency explicit. Plan said "verbatim copy" — adapted during execution because verbatim wouldn't compile. Commit `188bcfb`.
- **`verifyEmail` signature simplified.** Original's `|| editedLead.website` fallback inside the function was dead code — the only caller already passed `editedLead.website` explicitly. Removed.
- **Buggy parser behavior characterized as tests, not fixed.** Two tests assert that the 5-pattern fallback can override the structured-block name/title with adjacent title-keyword text. Improvements deferred per spec §6 R5; this refactor is structural, not behavioral.
- **Task 2: `parseResearchResult` call site updated to pass `enrichKeyPerson` explicitly** (commit `311e164`). The plan said "no changes at use sites" but verbatim deletion would have silently broken the keyPerson-fallback path because the inline closure captured `enrichKeyPerson` and the extracted helper takes it as a param. Decision logged here (see also the Task 1 decision above) — plan-vs-execution divergence is acceptable when the plan is provably wrong.
- **Task 2: `{{...}}` literals in JSX placeholder/help text are kept** (2 hits remain). They are user-facing hints inside `<textarea placeholder="...">` and an empty-state `<div>`, not substitution code. Plan's grep check was meant to catch leftover replacement loops, not docstring text.
- **Task 3: `setEditedLead` is exposed from `useLeadEdit`** (commit `aefd434`). Plan listed it in the hook's return but didn't include it in the destructure example. 4 enrich-flow call sites (`setEditedLead(updatedLead)` at the old lines ~291/340/360/558) still need to write the AI-found data back. Removing them now would change behavior; they collapse into `useLeadEnrichment` in Task 4 via dependency injection. Until then, the hook is the single owner and the container holds a destructured pass-through.
- **Task 3: enrich-fields prop-sync effect kept in container, not moved into `useLeadEdit`.** The original `useEffect` did 4 things — sync `editedLead` + 3 enrich fields. The plan suggested splitting it. Done: the `editedLead` sync now lives inside `useLeadEdit`; the 3 enrich-field syncs stay in the container until Task 4 moves them into `useLeadEnrichment`. Comment in code reflects this transitional state.
- **Task 4: `useLeadEnrichment` signature took `onSave` in addition to `(lead, editedLead, setEditedLead)`** (commit `594f7e6`). Plan listed only the first 3 params. The verbatim handler bodies for `handleEnrich`, `handleApproveEmail`, `handleSaveEnrichment` all call `onSave(updatedLead)` to persist; without it the hook would either drop persistence or need the orchestrator to wrap each handler. Adding the param is the minimal, explicit fix. Plan-vs-execution divergence acceptable because the plan was provably incomplete.
- **Task 4: `enrichCity` / `setEnrichCity` not destructured at the call site.** Both are still owned by the hook (used inside `handleEnrich` to feed Vertex AI), but the JSX never had a city input bound to either — neither read nor write. Destructuring them at the orchestrator would just trip the unused-vars warning. Hook remains the single owner; the destructure list documents what the JSX actually needs.
- **Task 4: 4 unused imports cleaned up in `LeadDetail.tsx`.** With `handleEnrich` moved into the hook, `VertexAiService`, `extractDomain`, `verifyEmail`, and `parseResearchResult` are no longer referenced from `LeadDetail.tsx`. Removed. `GeminiService`, `leadsApi`, `mapLeadFromDB`, `applyTemplatePlaceholders` stay — they're still used by the email handlers (Task 5 will move those out next).
- **Task 4: rate-limit countdown smoke not exercised.** Triggering it requires a real rate-limit response from the Vertex API, which the local env doesn't have credentials for. The countdown effect itself was a verbatim copy and is covered by the typecheck pass + visual code review. Defer behavioral smoke until a session with API keys.
- **Task 5: `useLeadEmail` signature added `setEditedLead` + `onSave` beyond plan's `(lead, activeTab)`** (commit `95e92be`). Same shape as Task 4 — `handleSendEmail` calls both at lines `setEditedLead(mappedLead); onSave(mappedLead);` after a successful send. Plan was provably incomplete; the divergence is logged for consistency.
- **Task 5: explicit `EmailTemplateAttachment` annotation on one `.map(att => ...)` callback.** Of the two `(template.attachments || []).map(...)` call sites in the hook, TS strict only complained about the one inside `loadEmailTemplates` (the other in `handleTemplateChange` was inferred fine — context-dependent). Cleaner to annotate the single offending callback than to add a marker; the original LeadDetail.tsx had `@ts-nocheck` so this surfaced only after the move.
- **Task 5: 6 imports removed from `LeadDetail.tsx`.** All email handlers + the rate-limit `useEffect` left the file, so `useEffect`, `EmailTemplate`/`EmailReply` types, `GeminiService`, the four named imports from `apiService` (incl. the always-unused `emailLogsApi`), `mapLeadFromDB`, and `applyTemplatePlaceholders` all became dead weight. Removed in the same commit.

### Strict-debt cleanup (2026-05-07)

- **`docx` shading: bug fix, not type-only.** `val: 'clear'` was being silently ignored at runtime (`createShading` destructures `{ fill, color, type }`, never reads `val`). Fixed to `type: ShadingType.CLEAR`. Commit `feb21c7`.
- **`EmailTemplateAttachment.template_id` made optional.** Smallest change that solves the type error: server fills the field on create/update; frontend never reads it. Cleaner than the `Omit<..., 'template_id'>` payload type the original marker suggested.
- **`vite.config.ts` typed config const.** Extracted async function to a `UserConfigFnPromise`-typed const so TS picks the correct `defineConfig` overload — no marker, no override.
- **`emailReplies.ts` typed payload.** `null` → `undefined` / omit. The model's `|| null` fallback already handles undefined optional fields at insert time, so behavior unchanged.

### Tooling / pattern decisions (older)

- **Pre-push hook is the CI substitute** (chosen during sub-project #1). No external CI yet. Solo + commits-on-main means this is the only gate.
- **Cleanup outright over `archive/` folder.** Decided in sub-project #1 — git history preserves deleted files; folders accumulate cruft.
- **Type-level edits only during strict-mode wave.** No logic refactoring. Where structural changes were needed, `@ts-expect-error TODO(refactor)` markers + `STRICT_DEBT.md` entries (sub-project #1 discipline).
- **AI provider abstraction: prompts extracted, providers scaffolded.** Route handlers still call SDKs directly because endpoints have provider-specific config (Google Search tools, json_object mode, system messages). Wrappers ready for future when those concerns normalize.

---
> Source: [lanfurama/ariyana-event-intelligence-crm](https://github.com/lanfurama/ariyana-event-intelligence-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
