## v0-vibe-assistant-ui

> **Read this entire file before every session. Non-negotiable.**

# VIBE — Agent Operating Rules

**Read this entire file before every session. Non-negotiable.**

Last reviewed: 2026-04-15 — rewritten from the previous version after drift incidents traced to the previous file being deleted and rules going unenforced. If you are a Claude Code session, a planning session, or any other agent, and you have not read this file top to bottom in this session, stop and read it now.

---

## 1. Current Position

**Active sprint:** v7.1 build — three parallel tracks: (1) recommendation mode in edge function, (2) Build tab UI at `/build` route, (3) Salesforce + Slack connectors live + production validation of one end-to-end autonomous loop. Governing doc is `NORTHSTAR_v7_1.md`. v7.0 + Addenda superseded.
**Blocking:** Closing the autonomous loop end-to-end in production. Code path exists (autonomous_executions, processor, edge function autonomous detection) but no production-validated execution with customer-visible recommendation output. Edge Function currently routes autonomous mode to dashboard fast path — wrong shape, must route to new recommendation mode.
**Next to ship:** Recommendation mode in edge function (target Apr 22), then Build tab internal release behind feature flag (target Apr 25). See `NORTHSTAR_v7_1.md` Section 6 for full calendar.
**Last updated:** 2026-04-15

If any of these three fields is older than one week, stop the session and ask Bill to update them before proposing any work. A stale current-position field is the most common source of sessions that propose the wrong thing confidently.

---

## 2. Hard Stops — Things That Will Not Change Without Bill's Explicit Approval

Every rule in this section exists because something broke. The reason is load-bearing. If you read a rule and don't understand why it's a rule, ask before working around it.

### 2.1 Dashboard fast path is locked

**File:** `apps/api/src/index.ts`, lines approximately 808–839
**Behavior:** When `mode === 'dashboard'`, the job handler bypasses the planner and makes a single `edgeCall({ prompt, model, mode: 'dashboard' })` that returns a single-file HTML output.
**Why locked:** This exact pipeline produced the Advanced Decisions demo successfully. Every prior attempt to "improve" it has broken the demo pipeline and cost a half-day to a full day of repair.
**Allowed changes:** None without Bill's explicit approval in writing. New dashboard capabilities ship as new modes, not as modifications to this path.
**Enforcement status:** Discipline only. Needs CODEOWNERS gate. See Section 4.

### 2.2 VIBE_SYSTEM_RULES is the only system prompt for the dashboard fast path

**File:** `supabase/functions/generate-diff/index.ts`, near line 581
**Behavior:** The edge function prepends `VIBE_SYSTEM_RULES` to every mode's system message. Dashboard mode adds `DASHBOARD_SYSTEM` and design-phase specs. No other system prompts, no prompt wrappers, no middleware prompt injection.
**Why locked:** Prompt stacking changes dashboard output in ways that pass code review but fail visual inspection. Every layer added has produced a dashboard regression.
**Allowed changes:** None without Bill's explicit approval. If you believe the system prompt needs to change, the first step is running the smoke test (Section 4) against the proposed change and attaching the output to the PR.
**Enforcement status:** Discipline only. Needs CI smoke test gate on any PR touching this file.

### 2.3 Single-file HTML output for dashboard mode

**Contract:** The edge function returns `{ diff: "<html>..." }` — one self-contained HTML file. `index.html` is the only file written. `manifest.json` contains `["index"]`. No `<a>` navigation between pages, no multi-page routing.
**Why locked:** Multi-page dashboards were attempted and broke preview rendering. The single-file output is what the preview iframe expects.
**Allowed changes:** None without a new mode. Multi-page dashboards, if built, are a separate mode.
**Enforcement status:** Discipline only. Needs output shape check in CI.

### 2.4 User prompt passes directly, no rewriting

**Contract:** The `enrichedPrompt` (user prompt + org context) is sent as the user message. No rewriting, no summarization, no template wrapping of the user's intent.
**Why locked:** Prompt rewriting was attempted to "improve" dashboard quality and broke the Advanced Decisions demo. Reverted in PR #514.
**Allowed changes:** None without explicit approval. Context injection via the kernel is fine; rewriting the user's text is not.
**Enforcement status:** Discipline only.

### 2.5 v7.1 is the active spec; v7.0 and prior addenda are superseded

**Rule:** Work is governed by `NORTHSTAR_v7_1.md`, not v7.0 or its addenda. v7.0, Trust Layer Addendum, Addendum B, and v6.0 Addendum are archived in `/docs/archive/` for audit trail only.
**What's in v7.1:** Three parallel tracks — recommendation mode (edge function output shape), Build tab UI (single user surface for both autonomous outputs and manual builds), connector expansion + production validation (Salesforce + Slack live, one autonomous loop demonstrated end-to-end). Target: May 9.
**What's deferred to v7.5+:** File sharing, cross-team feed cascades, marketplace switching costs, approval workflow UI, multi-page dashboards, additional connectors beyond Salesforce + Slack, Claude Agent SDK adoption, Managed Agents adoption, sub-agent splits.
**What's parked:** Go-to-market activities. See `GTM_PARKED.md` for the full list and restart conditions. Patents continue (Gary owns) and existing customer support continues.
**Enforcement status:** Discipline only. Any PR that proposes work outside v7.1 scope must say so in the PR description and reference the relevant section of `NORTHSTAR_v7_1.md` Section 7 (deferred) or `GTM_PARKED.md`.

### 2.6 Repositories and branches

- **Source of truth:** `UbiGrowth/VIBE`. All work merges here.
- **Do not push to:** `lupobill-rgb/v0-vibe-assistant-ui` — Vercel legacy, not maintained.
- **Claude Code branches:** `claude/*` with random suffix. Check `git branch -r` for exact names before merging.
- **Merges to main:** Happen from Bill's local PowerShell, not from Claude Code's git proxy. The proxy does not reliably reach the GitHub remote; always verify with `git fetch origin` + `git log origin/main` from local PowerShell before assuming a commit landed.

### 2.7 Do not deploy to production Vercel without instruction

All work deploys to `vibe_staging` only. Do not create PRs targeting the production Vercel project. Do not modify production Vercel project settings. Production deploys are Bill's decision, not an agent's.

### 2.8 This file itself

**Rule:** `CLAUDE.md` and `DEPLOY_CHECKLIST.md` may not be deleted, renamed, or moved without explicit approval. Any PR that deletes either file fails the pre-merge gate.
**Why:** The previous CLAUDE.md was deleted silently and the consequences surfaced days later as dashboard regressions. Silent guardrail removal is the worst class of incident because the cause-to-symptom delay prevents recognition.
**Enforcement status:** Needs CI existence check. Ten lines of bash in a GitHub Actions step. Highest-priority automation item.

---

## 3. Dashboard Fragility Map

Every path in this list can break dashboards even if the dashboard fast path itself is untouched. Changes to any of these require the smoke test (Section 4) before merge.

| Path | How it breaks dashboards |
|---|---|
| `apps/api/src/index.ts` lines 808–839 | Direct modification of the fast path — see 2.1 |
| `supabase/functions/generate-diff/index.ts` | Shared edge function — any change to `VIBE_SYSTEM_RULES`, `DASHBOARD_SYSTEM`, or design-phase spec affects dashboard output |
| `skill_registry.html_skeleton` column | Skeletons are dashboard templates. A skeleton without `new Chart()` calls renders blank charts. A skeleton with wrong tab overflow or sidebar z-index renders broken layouts. |
| **Render pipeline transformations** (`injectSupabaseCredentials`, `fast-paths.handler.ts` brand-token chain, `building/[id]/page.tsx` `buildPreviewHtml`, iframe `srcDoc` contract) | The HTML in the DB can be perfect but still render broken. Placeholder regex replacement, Chart.js load timing, iframe CSP, and handler output consistency are all transformations between the DB row and the browser. The skeleton verification query (Section 4.4) does not catch these. See Section 4.7 for the gates. |
| `supabase/migrations/*.sql` | Migration files merged but not applied produce schema drift. Dashboards query columns that don't exist in the live DB. |
| RLS policies on `jobs`, `projects`, `teams`, `team_members` | Tightened policies can break the `jobs → projects → teams → auth.uid()` chain that dashboard queries depend on. Dashboards return empty rows for authenticated users. |
| `apps/api/src/starter-site.ts` | `INITIAL_BUILD_BUDGETS` and `DASHBOARD_BUILD_BUDGETS` control timeout and concurrency. Changes to `buildConcurrency` can break against Supabase Edge Function concurrency limits. |
| `litellm/litellm_config.yaml` and `apps/executor/src/llm-failover.ts` | LLM routing changes alter which provider serves the dashboard prompt. Different providers produce different outputs. A `[LLM-FALLBACK]` in Railway logs during a smoke test is a merge blocker. |
| Context injector (`context-injector.ts`) | Changes to which skills, design tokens, or brand tokens get injected change dashboard output even when the edge function and fast path are untouched. |
| `VIBE_SAMPLE` data state default | The sample data state is the default for dashboards. Changes to how sample data is generated or fabricated change what every dashboard displays. |

**The rule:** A change to any row in this table is a change to dashboards, regardless of what the PR description says. If the smoke test is not run on a PR that touches any of these, the PR is not ready to merge.

---

## 4. Pre-Merge Gates

No PR merges to main without passing every gate in this section. Gates marked "CI" are enforced automatically. Gates marked "human" require a session to perform them and paste proof in the PR description.

### 4.1 CLAUDE.md and DEPLOY_CHECKLIST.md exist

**Status:** Needs CI.
**Check:** `test -f CLAUDE.md && test -f DEPLOY_CHECKLIST.md`
**Action on fail:** Block merge until files are restored.

### 4.2 Smoke test — dashboard build

**Status:** Human until automated.
**Prompt:** `show me my pipeline`
**Workspace:** Sales team
**Checks that must pass:**
- [ ] Charts render on first load (no follow-up prompt needed)
- [ ] Navigation links respond to clicks
- [ ] Buttons respond to clicks
- [ ] Token count under 30,000 (Railway logs)
- [ ] No `[LLM-FALLBACK]` in Railway logs
- [ ] No `[QA REASONS]` repair pass triggered
- [ ] No raw "html" text visible in preview

**When required:** Any PR touching any row in the Dashboard Fragility Map (Section 3).
**Proof:** Paste the test job ID and a screenshot or description of the rendered output into the PR description under "Smoke Test Results."

### 4.3 Migration applied before merge

**Status:** Human until automated.
**Rule:** No migration file merges to main unless it has already been applied to the live Supabase DB via MCP `apply_migration`.
**Sequence:** Apply to DB → verify via `execute_sql` → commit migration file for audit trail → merge PR.
**Proof:** Paste the `apply_migration` result and a verification query result in the PR description.
**Why:** 5+ migration files were merged to GitHub without being applied to DB, causing schema drift and silent dashboard failures. This is the highest-frequency root cause of "dashboards broke for no reason."

### 4.4 Skeleton verification query

**Status:** Human until automated.
**When required:** Any PR that adds or modifies `skill_registry.html_skeleton`.
**Query:**

```sql
SELECT skill_name,
  length(html_skeleton) as length,
  (LENGTH(html_skeleton) - LENGTH(REPLACE(html_skeleton, 'new Chart(', '')))
    / LENGTH('new Chart(') as charts,
  html_skeleton LIKE '%__VIBE_TEAM_ID__%' as has_team_token,
  html_skeleton LIKE '%window.__VIBE_SAMPLE__%' as has_sample_data,
  html_skeleton LIKE '%try{%vibeLoadData%' as has_try_catch,
  (html_skeleton !~ '\.tabs[^}]*\{' OR html_skeleton ~ '\.tabs[^}]*overflow-x\s*:\s*auto') as tabs_overflow_ok,
  (html_skeleton !~ '\.tabs-bar[^}]*\{' OR html_skeleton ~ '\.tabs-bar[^}]*overflow-x\s*:\s*auto') as tabs_bar_overflow_ok,
  (html_skeleton !~ '\.sidebar[^}]*\{' OR html_skeleton !~ '\.sidebar[^}]*z-index:\s*5\d') as sidebar_z_ok
FROM skill_registry
WHERE skill_name = '[skill_name]';
```

**Quality gate — all must be true:**
- `length > 20000`
- `charts >= 4`
- `has_team_token = true`
- `has_sample_data = true`
- `has_try_catch = true`
- `tabs_overflow_ok = true`
- `tabs_bar_overflow_ok = true`
- `sidebar_z_ok = true`

**Proof:** Paste the query result in the PR description. `charts = 0` is an automatic reject. Do not merge and do not proceed to the next task.

### 4.5 Locked files untouched

**Status:** Needs CODEOWNERS.
**Rule:** No PR may modify `apps/api/src/index.ts` lines 808–839, `supabase/functions/generate-diff/index.ts`, or this file without explicit approval from Bill.
**Action on fail:** Block merge.

### 4.6 Constraint values verified

**Status:** Human.
**Rule:** Before writing any `jobs.execution_state` or `job_events.severity` value in code, verify the value is in the allowed CHECK list via:

```sql
SELECT pg_get_constraintdef(oid) FROM pg_constraint WHERE conname = '...';
```

**Allowed `jobs.execution_state`:** `queued`, `cloning`, `building_context`, `calling_llm`, `applying_diff`, `running_preflight`, `creating_pr`, `completed`, `complete`, `failed`, `planning`, `building`, `validating`, `testing`, `security`, `qa`, `ux`
**Allowed `job_events.severity`:** `info`, `error`, `success`, `warning`, `warn`

### 4.7 Render Pipeline Safety — non-negotiable

These rules exist because between 2026-04-14 09:00 and 22:00, seven PRs (#596–#602) shipped to fix "charts not rendering in templates" — each necessary, each insufficient on its own. The HTML in `skill_registry.html_skeleton` passed every existing verification gate the entire time. The bugs were in the render pipeline between the DB row and the browser. Each rule below is tied to one of those merged fixes.

**Rule 1 — Placeholder safety.** Never use `__SUPABASE_URL__`, `__TEAM_ID__`, or `__VIBE_TEAM_ID__` as JavaScript identifiers. They get mangled by the global regex replacement into invalid syntax (`const https://...`) and throw `SyntaxError: Missing initializer`. Canonical local variable names: `VIBE_SB_URL`, `VIBE_SB_KEY`, `VIBE_TEAM_ID`. Files: any skeleton or template that emits JS referencing these placeholders.

**Rule 2 — Chart.js load timing.** Never call `Chart.defaults.*` or `new Chart` at script-parse time. Wrap in a 50ms poll with a 10s cap that waits for `typeof Chart !== 'undefined'`, then calls `boot()`. On timeout, still call `boot()` so KPI and table rendering runs even without charts. Files: skeleton bootstrap blocks, any inline script that uses Chart.js.

**Rule 3 — Preview iframe contract.** The iframe in `apps/web/app/building/[id]/page.tsx` MUST use `srcDoc`, not `src=blob:`. Blob-origin iframes block CDN script loads under the default CSP. Sandbox attributes: `allow-scripts allow-same-origin`. File: `apps/web/app/building/[id]/page.tsx`.

**Rule 4 — Handler output consistency.** Deterministic template, dashboard, and app fast-path handlers must pass the credentials-replaced HTML to BOTH `fs.writeFileSync` AND `storage.setTaskDiff`. The frontend reads from the task diff, not from disk. The audit log still receives raw pre-creds HTML. Files: `apps/api/src/index.ts` (deterministic-template path), `fast-paths.handler.ts` (dashboard and app fast paths).

**Rule 5 — Placeholder replacement sites.** Every site that runs the `__SUPABASE_URL__` regex must also replace `__VIBE_TEAM_ID__` alongside the legacy `__TEAM_ID__`. There are three sites: `index.ts` `injectSupabaseCredentials`, `fast-paths.handler.ts` brand-token chain, and `building/[id]/page.tsx` `buildPreviewHtml`. Adding a fourth site without updating it is a regression on this rule.

**Rule 6 — Runtime smoke check.** The DB verification query from Section 4.4 (Skeleton Verification) is necessary but NOT sufficient. After any change to templates, the generator, fast-paths/planner/dashboard handlers, `injectSupabaseCredentials`, or `buildPreviewHtml`, run a live build and verify five runtime checks in the iframe devtools console:
- No `SyntaxError: Missing initializer` errors
- No CSP violation errors blocking script loads
- `typeof Chart !== 'undefined'` after page load
- KPI cards populated with values (not loading skeletons)
- At least one chart rendered with data

Paste the results in the PR description.

**Why these rules exist:** Each rule names the files it touches so future sessions can find the canonical patterns without grepping the whole repo. The skeleton verification query in Section 4.4 only checks the HTML in the DB. These rules check the rendered output in the browser. Both gates are necessary. Skipping these gates is what produced the seven-PR firefight on 2026-04-14.

**Enforcement status:** Discipline only. Needs CI gate that runs a headless browser smoke test on PRs touching the file list above. Highest-priority CI item after the existence checks (Section 4.1).

---

## 5. The Closed Runtime Loop

As of 2026-04-13, VIBE operates on a closed runtime loop. This is the structural threshold Sprint 1 and Sprint 3 crossed: authenticated integrations are operational assets, queryable at runtime, through a real planner/worker path, with live data, failover protection, and spend controls.

A loop is only "closed" if ALL of the following are true on every execution path:

1. **Authenticated identity resolves correctly** — no cross-tenant ambiguity
2. **Connection is selected deterministically** — no fallback guesswork
3. **Planner emits correct mode** — runtime vs build is not inferred later
4. **Worker executes without human intervention** — no manual retries
5. **Data is fetched from live system** — no mocked or cached stand-ins
6. **Output renders in-product** — no logs-only success
7. **Spend is bounded** — no silent overages
8. **User is notified on constraint** — no hidden failures

If any one of these breaks, the loop is OPEN. An open loop is a regression, regardless of what feature it enabled.

### 5.1 Enforcement

- **No demos on non-runtime paths.** Every demo between now and revenue runs through the closed loop.
- **No features that bypass planner/worker.** If a change skips the orchestrator, it reintroduces an asterisk.
- **No connector work that doesn't hit `NangoService.fetchRecords`.** Universal dispatch is the only supported runtime path. Bespoke connector adapters are forbidden.
- **No UI work that assumes data without runtime fetch.** If a page renders `__VIBE_SAMPLE__` in production, the loop is open there.

### 5.2 PR description requirement

Every PR description must state which of the 8 conditions the change touches, and confirm none are weakened. Silence is not acceptance.

---

## 6. Stack and Key Files

### 6.1 Stack

- **Frontend:** Next.js on Vercel, deploys from `UbiGrowth/VIBE` main
- **API:** NestJS on Railway (`@vibe/api`, port 8080)
- **Database:** Supabase project `ptaqytvztkhjpuawdxng`
- **LLM:** Claude primary. GPT-4 circuit-breaker fallback on 429/529/timeout ONLY. Never a quality fallback.
- **Connectors:** Nango. All connector code must be fully provider-agnostic — never reference HubSpot or any specific provider in code or prompts.

### 6.2 Key files

| Path | What it does |
|---|---|
| `apps/api/src/index.ts` | Job routing, `edgeCall`, `POST /jobs` handler. Contains the locked dashboard fast path at lines 808–839. |
| `apps/api/src/starter-site.ts` | `INITIAL_BUILD_BUDGETS`, `DASHBOARD_BUILD_BUDGETS`, `mapWithConcurrency` |
| `apps/api/src/storage.ts` | `ExecutionState` and `EventSeverity` type definitions |
| `apps/api/src/connectors/autonomous-processor.service.ts` | Autonomous execution dispatcher. v7.0-adjacent — touch only if current sprint position allows. |
| `supabase/functions/generate-diff/index.ts` | Edge Function. Source of truth for LLM behavior. Contains `VIBE_SYSTEM_RULES`. |
| `apps/web/app/chat/page.tsx` | Frontend job submission |
| `apps/web/app/building/[id]/page.tsx` | Preview iframe host. Contains `buildPreviewHtml`. Section 4.7 Rule 3 governs the iframe `srcDoc` contract. |
| `apps/api/src/fast-paths.handler.ts` | Dashboard and app fast-path handlers. Section 4.7 Rule 4 governs handler output consistency. Brand-token chain is one of the three placeholder replacement sites in Rule 5. |
| `injectSupabaseCredentials` (in `apps/api/src/index.ts`) | One of the three placeholder replacement sites in Section 4.7 Rule 5. |
| `context-injector.ts` | Kernel context injection, `resolveDepartmentSkills()` |

### 6.3 Build budgets

- `INITIAL_BUILD_BUDGETS.maxWallTimeMs`: 420,000 (7 min)
- `INITIAL_BUILD_BUDGETS.stepDeadlinesMs.building`: 360,000 (6 min)
- `INITIAL_BUILD_BUDGETS.buildConcurrency`: 3 (overridden to 1 in `index.ts` — pages build sequentially ~85s each)
- Do not change concurrency without testing against Supabase Edge Function concurrency limits.

---

## 7. Session Rules

1. **Reliability over cleverness.** A working MVP beats clever broken code.
2. **Chunked mode.** ONE file per diff. MAX 200 lines per diff. Ask for exact path first if unsure.
3. **Read before write.** Always read the target file before making any change.
4. **No large refactors** unless cleanup mode is explicitly triggered by Bill.
5. **RLS on.** Secrets never in logs or LLM context. Ever.
6. **Every change traceable:** job → diff → log → test result.
7. **OSS first.** No custom primitives when a library already exists and is proven.
8. **Scan before planning.** Query live schema and `git log origin/main` before proposing any sprint.
9. **Verify constraint values** before writing any `execution_state` or `severity` value. See Section 4.6.
10. **One concern per session.** Do not mix fixes and features. Do not mix instrumentation and refactor.
11. **Define DONE WHEN before writing code.** A session without a DONE WHEN drifts.
12. **Revert, don't layer.** Regressions get reverted, not patched over.
13. **`INDEX.TS GATE`:** If a file exceeds 800 lines, extract before adding new logic.
14. **Delete branches after every merged PR.** Run `git remote prune origin` weekly.
15. **Never prompt-engineer workarounds** that paper over real platform gaps.

---

## 8. What NOT to Build Right Now

This section exists because sessions keep proposing plausible-looking work that is not the current priority. Each item has a reason and a gate. The list is updated when v7.1 progresses or when scope changes.

**Build tab is NOW IN SCOPE per v7.1 Track 2.** It moved out of this list on 2026-04-15. Reference: `NORTHSTAR_v7_1.md` Section 4 Track 2.

Currently deferred:

- **Claude Agent SDK adoption for `/jobs`.** The SDK is the right long-term answer for the agent loop, but migrating `/jobs` is a rewrite of the most important file in the backend. Gate: INDEX.TS GATE triggers an extract for unrelated reasons, or a specific capability gap forces the migration.
- **Managed Agents adoption.** Potentially the right answer for long-horizon autonomous work and PR verification. Gate: docs read, pricing and residency and compliance answered, memo written. Do not start until memo exists.
- **Planner / coder / verifier sub-agent split.** The severity labels exist in schema but the sub-agents don't. Do not build real sub-agents. Use phase labels instead. Gate: customer asks for separable agent inspection in writing.
- **Approval hooks UI.** Schema exists in Trust Layer (approval_signatures). UI does not. Gate: a specific customer requirement names the approval flow OR autonomous loop in production reveals a need for human-in-loop approval before action execution.
- **Multi-audience UI renderers.** Marketing / ops / dev views. The Build tab v7.1 ships as one view with progressive disclosure. Gate: a specific customer segment complains about the single view after v7.1 ships.
- **File sharing within the platform (Layer 5).** Mentioned in earlier scope discussions but deferred. Gate: v7.1 ships AND a specific customer requirement names the file-sharing use case.
- **Cross-team feed cascades** (autonomous execution in one team triggers another team's skill). Gate: v7.1 autonomous loop proven for single-team case first.
- **Marketplace switching-cost layer development.** Routes exist, real switching costs do not. Gate: v7.1 ships. Then evaluate whether marketplace expansion is the right Q3 priority.
- **Custom diff engine, custom feed component, React Flow DAG viz, n8n wrapper, custom workflow engine.** All of these are available as proven libraries or are not needed at all. Gate: the library evaluation rejects all available options with specific reasons.
- **Dispatch-phase event instrumentation** (lines 13–98 of `autonomous-processor.service.ts`). Low value, requires schema changes. Gate: post-project-resolved instrumentation proves valuable in production first.
- **Second event system, second run state machine, second runtime.** VIBE owns the spine. Never build a parallel system when extension is possible.
- **Additional connectors beyond Salesforce + Slack** (GA4, Mixpanel, Snowflake, PostgreSQL, BigQuery, AWS S3). Stay as Nango stubs until v7.1 ships. Gate: v7.1 done AND specific customer requires the connector.
- **All GTM activities listed in `GTM_PARKED.md`.** Restart conditions documented there.

---

## 9. Never

- Silent failures. Always return plain-English explanation plus next action.
- Raw stack traces to users.
- Customer API keys. All LLM calls through our accounts.
- Whole-file rewrites. Diffs only.
- Push to `lupobill-rgb` for bug fixes — always `UbiGrowth/VIBE`.
- Manual schema changes outside migrations. Audit `information_schema.columns` before writing any migration.
- Hardcoded user IDs, team IDs, or service tokens in code. All must come from env or auth context.
- References to specific providers in connector code. Provider-agnostic only.

---

## 10. Governing Documents

Priority order. Higher = more authoritative. Lower documents are superseded by higher ones on any conflict.

1. **CLAUDE.md** (this file) — session enforcement
2. **NORTHSTAR_v7_1.md** — strategic direction (Autonomous Company OS, current spec)
3. **GTM_PARKED.md** — what's parked and restart conditions
4. **DEPLOY_CHECKLIST.md** — blast radius gate
5. **VIBE_Design_System_Spec.md** — Figma-quality output standard

**Archived (reference only, not authoritative):**
- `/docs/archive/VIBE_NorthStar_v7_0.docx` — superseded by v7.1
- `/docs/archive/NORTHSTAR_V7_ADDENDUM_B.md` — superseded by v7.1
- `/docs/archive/VIBE_NorthStar_v7_0_Trust_Layer_Addendum.docx` — Trust Layer shipped, addendum no longer active
- `/docs/archive/VIBE_NorthStar_v6_0_Addendum.docx` — superseded by v7.1
- `/docs/archive/NORTH_STAR.md` (v4.0 March 20) — superseded
- `/docs/archive/VIBE_Revenue_Sprint_Prompts.md` — Revenue Sprint sequence superseded by v7.1 Track structure

---

## 11. Billing

- `STRIPE_SECRET_KEY` → Railway env only. Never Vercel. Never frontend.
- `STRIPE_WEBHOOK_SECRET` → Railway env only.
- Pricing model: Seat fee ($17/user/month) + token consumption (750K included/user/month).
- Free trial: 30 days full access, no credit card required.
- Volume discounts: 500+ users $15, 2500+ $12, 10000+ $10.
- Builder persona tiers (Starter / Pro / Growth / Team / Portfolio / Enterprise) retained for feature gating.
- Cost rates: read from DB `cost_rates` table, never hardcoded.
- Customer-facing copy: no mention of markup, tokens, or LLM.

---

## 12. Design System

Every build output must follow `DESIGN_SYSTEM_RULES`. Injected by `context-injector.ts` after department skills, before user prompt. Brand tokens from the kernel drive color palette. Light/dark follows user intent.

Skeleton design standard: Dark theme `#0A0E17`, Space Grotesk headings, Inter body, `#00E5A0` primary. All regulated industry skeletons (pharma, finserv, healthcare) must include an ISO 27001 panel.

---

## 13. Changelog

| Date | Author | Change |
|---|---|---|
| 2026-04-15 | Bill + Claude (planning session) | Full rewrite after previous file was deleted and dashboard regressions traced to guardrail absence. Reorganized rules by failure mode. Added Dashboard Fragility Map. Added explicit "What NOT to Build Right Now" section. Marked enforcement status on every gate. Added CI existence check as highest-priority automation item. |
| 2026-04-15 | Bill + Claude (second pass) | Filled in Section 1 Current Position from repo scan (Sprint 3 — closed runtime loop hardening, dashboard path repair). Reconciled Section 2.5 — Reactive Kernel completion is in scope; further autonomous v7.0 features remain gated. Updated Section 8 Build tab gate to reflect dashboard stability as the real blocker, not "Revenue Sprints complete." |
| 2026-04-15 | Bill + Claude (third pass) | Merged 6-rule RENDER PIPELINE SAFETY section from orphan branch `claude/claude-md-render-pipeline-rules-zuq00` (commit b9755cf, originally authored 2026-04-14 after the seven-PR firefight on PRs #596–#602). Added as Section 4.7. Updated Section 3 Dashboard Fragility Map with render pipeline row. Added render pipeline files to Section 6.2 Key Files table. Orphan branch can be deleted after this version commits to main. |
| 2026-04-15 | Bill + Claude (fourth pass) | Aligned with NORTHSTAR_v7_1.md. Section 1 updated to v7.1 build (three parallel tracks: recommendation mode, Build tab, connector expansion). Section 2.5 rewritten — v7.1 supersedes v7.0 + addenda. Section 8 updated — Build tab moved out of "don't build" list (now in scope per Track 2), all GTM items moved to GTM_PARKED.md reference. Section 10 governing docs updated — v7.1 is the active strategic document, prior v7.0 docs archived. |

---

## 14. The Meta-Rule

Every rule in this file is either enforced by CI or enforced by discipline. The rules enforced by discipline are a backlog: they should move to CI as soon as possible, because discipline rules fail silently when a session forgets, and that silent failure is the root cause of the dashboard regression cycle this file exists to prevent.

If you are reading this and a rule marked "Discipline only" has a CI-enforceable equivalent you can ship in the current session, ship it. Converting a discipline rule to a CI rule is always in scope, even during other work.

---
> Source: [lupobill-rgb/v0-vibe-assistant-ui](https://github.com/lupobill-rgb/v0-vibe-assistant-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
