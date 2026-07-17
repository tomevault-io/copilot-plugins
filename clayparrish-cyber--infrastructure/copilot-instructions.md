## infrastructure

> Registry-driven, multi-project agent infrastructure. Runs nightly via GitHub Actions, Supabase is the source of truth. Reviews 6+ codebases automatically for security, bugs, UX, content, polish, and performance issues.

# Infrastructure Project Context

## What This Is

Registry-driven, multi-project agent infrastructure. Runs nightly via GitHub Actions, Supabase is the source of truth. Reviews 6+ codebases automatically for security, bugs, UX, content, polish, and performance issues.

## Architecture

```
.github/workflows/
└── nightly-review.yml          # 6-job pipeline (runs 2am CST Sun/Mon/Wed/Fri)

agents/
├── registry.json               # Project + agent config (22 agents, automation profiles, schedules)
├── config/
│   └── credentials.json        # Token/key registry with expiry dates
├── orchestrator/
│   └── decide-roster.md        # AI orchestrator prompt (builds nightly roster)
├── reviews/
│   └── {project}/              # Per-project review prompts (7 themes)
│       ├── security-review.md
│       ├── ux-layout-review.md
│       ├── bug-hunt-review.md
│       ├── content-value-review.md
│       ├── polish-brand-review.md
│       ├── performance-review.md
│       └── aso-retention-review.md
├── ops/
│   └── weekly-ops-combined.md  # Sunday ops + evaluator (combined)
├── workers/
│   └── work-loop-manager.sh    # Executes approved work items
└── lib/
    ├── collect-orchestrator-signals.ts
    ├── sync-to-supabase.ts
    ├── build-agent-context.ts     # Persistent memory context injection
    └── git-activity-scanner.sh

packages/agent-learning/
└── src/
    ├── scoring/signal-score.ts
    └── learning/similarity.ts
```

## Nightly Pipeline (6 Jobs, 4x/week: Sun/Mon/Wed/Fri)

1. **setup** — Determines today's theme + project scope from day-of-week
2. **clone-projects** — Clones relevant repos from registry
3. **orchestrator** — Commit gate checks git_activity; if all quiet, skips AI call and produces empty roster. Otherwise AI builds roster.json (cap: 5 entries, falls back to static if it fails)
4. **reviews** — Runs `claude -p` for each roster entry sequentially
5. **sync** — Writes findings to Supabase `work_items` + `agent_runs_v2`
6. **workers** — Executes approved work items via `claude -p`
7. **reconcile** — Stale item sweep + combined daily ops agent (credentials + health + brief in 1 call)

## Weekly Schedule (4x/week as of 2026-04-13)

| Day | Theme | Scope |
|-----|-------|------|
| Mon | security-review | core |
| Wed | bug-hunt-review | core |
| Fri | security-review + polish (odd weeks) | core |
| Sun | weekly-ops-combined + business-synthesis + portfolio-audit (1st Sun/mo) | all |

Tue/Thu/Sat: no pipeline runs. Specialist agents (competitive-intel, content-writer, creative-provocateur, marketing-analyst) moved to on-demand only.

## Automation Profiles

- **Core**: sidelineiq, airtip, dosie, glossy-sports, mainline-apps, mainline-dashboard
- **Scaffolded**: gt-ops, menu-autopilot, the-immortal-snail, gt-website
- **Ops-only**: infrastructure

## Agent Budgets (Updated 2026-04-13)

Target: ~$100-140/month (down from ~$450). Key savings: 4x/week schedule, commit gate skips quiet projects, specialists on-demand, consolidated ops agents.

## Key Tables (Supabase)

| Table | Purpose |
|-------|---------|
| `agents` | Roster: id, name, tier (1-4), status, budget_monthly |
| `agent_runs_v2` | Run history: agent_id, project, status, cost, tokens |
| `work_items` | All findings: status flow discovered→triaged→approved→in_progress→review→done |
| `work_item_events` | Audit trail |
| `agent_budget_summary` | View: per-agent monthly cost vs budget |

## Secrets (GitHub Actions)

- `ANTHROPIC_API_KEY` — Powers all `claude -p` calls. **If credits run out, entire pipeline silently fails.** Set auto-reload at console.anthropic.com.
- `GH_PAT` — Clones private repos
- `SUPABASE_URL` / `SUPABASE_SERVICE_ROLE_KEY` — Reads/writes findings
- `COMMAND_CENTER_URL` / `COMMAND_CENTER_API_KEY` — Posts to dashboard

## Known Gotchas

- **`claude -p` in while-read loops**: Must use fd 3 (`read -u 3`, `done 3<<<`) and `< /dev/null` to prevent stdin consumption. See skill: `claude-p-while-read-stdin-consumption`.
- **Static fallback**: If orchestrator AI fails, workflow falls back to naive static roster. Check `"source": "static-fallback"` in roster JSON.
- **Worker directory mismatch**: Workers look for projects at `projects/{name}` — monorepo subdirectories must be extracted correctly in clone step.
- **Budget view uses current month**: `agent_budget_summary` resets monthly. Runs from prior months don't count against current budget. View now includes `effective_budget` (base + overrides) and `budget_enforcement_mode` (observe/warn/enforce). All agents default to `observe` while calibrating.
- **Entity grouping**: `entities` and `project_entities` tables map projects to GT/Mainline/Personal. `entity_budget_summary` view rolls up costs per entity per month.

## Recent Changes

- **2026-04-21** — Worker silent-failure capture (PR #15 at clayparrish-cyber/infrastructure, open). Both worker failure paths in `work-loop-manager.sh` (empty claude -p output, timeout/crash) now capture `$output_file.stderr` (truncated to 4KB), exit code, and duration into `agent_runs_v2.error_details` — JSONB column already existed per `dashboard/supabase/migrations/20260206000000_autonomous_org_foundation.sql`, just never populated. Also de-silenced the `agent_runs_v2` INSERT: non-2xx HTTP responses are now logged locally instead of `> /dev/null 2>&1`-ed. Stderr now also surfaces in `work_items.execution_log` so CC dashboards show real diagnostics instead of hardcoded "Worker timed out or crashed" strings. Closes CC `d5ab142b`.
- **2026-04-13** — Pipeline cost optimization: ~$450/mo → ~$100-140/mo. Schedule 7x→4x/week (Sun/Mon/Wed/Fri). Commit gate skips orchestrator when all projects quiet. 4 specialist agents moved to on-demand. Capacity cap 8→5. Daily ops consolidated from 3 agents to 1 (`daily-ops-combined.md`). Sunday ops consolidated (`weekly-ops-combined.md`). Health-check job removed (folded into daily ops). Portfolio audit monthly. Managed pipeline schedule disabled (dry-run was costing $1-3/night).
- **2026-04-09** — nightly-review.yml fully migrated from GH_PAT → GitHub App auth. Created App "mainline-nightly-review" (ID 3331983) installed on all 7 project repos. PAT trap durably gone.
- **2026-04-09** — Marketing automation workflow moved out of infrastructure to clayparrish-cyber/mainline-apps (the repo that owns the worker code).
- **2026-04-01** — Fixed auto-triage gap: human-created work items now start at "triaged" instead of "discovered".
- **2026-03-17** — Agent infra upgrade: budget observability, entity grouping, persistent agent memory.

## Current Status (2026-01-23)

**Completed:**
- [x] 15 Phase 1 agents installed
- [x] inventory-health.ts and monday-briefing.ts verified working
- [x] GT-IMS Command Center deployed (gt-ops.vercel.app/command-center)
- [x] Executive Dashboard design document written
- [x] **Agent Learning Loop Closed** (Ralph mission completed):
  - Signal scores calculated on human decisions (60% decision, 20% recency, 20% priority)
  - Similarity query finds past recommendations before generating new ones
  - Few-shot examples formatted and included in recommendation reasoning
  - 7-day deduplication prevents duplicate recommendations
  - All 32 unit tests pass

**Next Steps:**
- [ ] Build Executive Dashboard MVP (Priority 2)
- [x] **Add prime directives to agents** (Priority 3) - ✅ COMPLETED 2026-01-23
- [ ] Fix hardcoded demand value in inventory-health.ts (Priority 4)
- [ ] Create legal-review.ts, research-request.ts, db-optimization.ts agent scripts

### @clayparrish/agent-learning Package

Shared npm package for agent learning capabilities:

```
packages/agent-learning/
├── src/
│   ├── scoring/signal-score.ts      # calculateSignalScore()
│   ├── learning/
│   │   ├── similarity.ts            # findSimilarRecommendations()
│   │   └── few-shot.ts              # formatExamplesForPrompt()
│   ├── embeddings/generate.ts       # generateEmbedding()
│   ├── directives/
│   │   ├── types.ts                 # Directive, DirectiveContext, Violation
│   │   └── check.ts                 # checkDirectives(), formatViolations()
│   └── db/connection.ts             # Database connection helpers
└── dist/                            # Compiled output (use this for imports)
```

### Agent Prime Directives (Added 2026-01-23)

Guardrails that prevent agent drift by flagging violations for human review:

**Config:** `config/agents/prime-directives.ts`

**Universal Directives (all agents):**
- `cite-source` - Must cite data source for every recommendation
- `flag-uncertainty` - Flag when confidence < 80%
- `human-reasoning` - Include human-readable reasoning (>20 chars)

**Agent-Specific Directives:**
- INVENTORY: `fresh-data` (data < 7 days), `realistic-demand` (CRM-based)
- LEGAL: `no-advice` (flag only, never provide legal advice)
- RESEARCH: `cite-sources` (must include URLs)

**Enforcement:** Soft - violations set `needsVerification=true` and populate `violationReason` for Command Center review.

**Usage in venture projects:**
```json
// package.json
"dependencies": {
  "@clayparrish/agent-learning": "file:../../infrastructure/packages/agent-learning"
}
```

**Note:** Next.js 16 requires webpack config for `file:` linked packages. See skill: `nextjs-exclude-scripts-from-build`

## Development

```bash
cd ${PROJECTS_DIR}/infrastructure

# Agent configs
cat config/agents/phase-1-config.ts

# View design docs
ls docs/plans/
```

## Related Projects

| Project | Path | Relationship |
|---------|------|--------------|
| GT-Ops | ${PROJECTS_DIR}/gt-ops | Primary test bed, has working agents |
| Menu Autopilot | ${PROJECTS_DIR}/menu-autopilot | Future agent integration |
| AirTip | ${PROJECTS_DIR}/menu-autopilot (at /tips) | Future agent integration |
| SidelineIQ | ${PROJECTS_DIR}/SidelineIQ/sideline-iq | Future agent integration |
| Dosie | ${PROJECTS_DIR}/Dosie/web | Future agent integration |


## Org Documentation Governance (3-Tier)
<!-- ORG_DOCS_3_TIER_POLICY -->

For non-trivial tasks, persist context to Markdown before finishing. Do not leave critical context only in chat.

Use one tier per note:
1. `canonical`: long-lived source of truth; requires `owner` and recurring `review_by`; `expires_at` should be `none`.
2. `operational`: medium-lived project context; requires both `review_by` and `expires_at`; default TTL 30 days.
3. `ephemeral`: short-lived scratch context; requires both `review_by` and `expires_at`; default TTL 7 days.

Required front matter on governed notes:

```md
---
type: canonical|operational|ephemeral
owner: <team-or-person>
created_at: YYYY-MM-DD
review_by: YYYY-MM-DD
expires_at: YYYY-MM-DD|none
source_of_truth: <canonical file/link/ticket>
status: active|superseded|expired
---
```

Preferred path: `docs/knowledge/{canonical|operational|ephemeral}`.
Archive stale/superseded notes to `docs/knowledge/archive` and update `status`.

Run `${PROJECTS_DIR}/scripts/check-stale-docs.sh` before major handoffs.

---
> Source: [clayparrish-cyber/infrastructure](https://github.com/clayparrish-cyber/infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
