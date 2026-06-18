## claude-todos-skill

> Per-project todo tracking with a relational project-knowledge brain — decisions, business rules, contradictions, stakeholders, scope changes — and a local broadsheet dashboard at localhost:5757. Use when the user wants to plan, sync, or close out implementation work in a project, capture a decision/rule/contradiction/feedback to remember later, or browse the dashboard.


# Todos Skill

Tracks implementation todos per project AND captures the relational project intelligence around them. Each project gets:

- `.claude/todos.json` — tasks and phases (committed to repo).
- `.claude/brain.db` — SQLite "project brain": stakeholders, feedback, decisions, contradictions, business rules, scope changes, architecture notes, **per-task plans + their planning Q&A** (committed to repo).
- `.claude/{docs,notes,diagrams}/` — long-form markdown and Mermaid diagrams referenced from `brain.db`.

A local Bun server at `http://localhost:5757` reads every tracked project and renders an aggregated newspaper-style dashboard with Today / Project / Activity / Brain / Brief / **Week** / **Hold** / Studio tabs.

## Commands

- **`/todos-init`** — bootstrap a project: find the PRD/scope doc, extract feature list, audit the codebase for what's already built, mark MVP/95%/100% stages, dependency-order everything, write `.claude/todos.json`.
- **`/todos-plan`** — batch-plan a scope of tasks up front (backend-first): walk the scoped tasks in dependency order, surface every ambiguity once, pin each task's contract, and print a runway report of how far the build can go autonomously before you're needed again. Writes `plan` + `plan_question` rows to `brain.db`; mirrors `plan_status`/`plan_id` onto the board.
- **`/todos-build`** — execute the planned tasks autonomously (backend-first): build each in dependency order off its pinned contract, run the backend suite, and halt + flag downstream the moment tests fail. Workflow-tool engine by default (isolated agent per task), `--mode subagent` for a watchable foreground run; checkpoint-commits per phase on a `todos-build/*` branch. Re-running resumes (done tasks skipped).
- **`/todos-brief`** — roll the built backend's pinned contracts into `.claude/docs/frontend-brief.md`: a deduped, render-ready manifest of entities, endpoints (with response shapes), and events, grouped by feature. The bridge from a finished backend to the frontend pass — the input `/todos-design` consumes. **`--from-code`** reverse-extracts the brief from an existing Laravel+Inertia app (a legacy / UX-rescue with no `/todos-plan` history) by reading routes + `Inertia::render` props + FormRequests + Resources.
- **`/todos-design`** — run the frontend design pass: turn `frontend-brief.md` into a built, dual-audited UI using the four design pillars in `lib/frontend/` (process / knowledge / distinctive / effortless-ux). Foreground + interactive — proposes a direction and confirms it with you, derives nav from the brief's feature groups, builds each feature's screens **composed from existing components**, then halts at two independent gates: craft ("would I sign my name to this?") and usability ("can a stranger operate it?" — zero sev-3/4 to ship). Renders + eyeballs the pixels, saves `.interface-design/system.md`. Systems/dashboards first; marketing briefs hand off to `taste-skill` / `/frontend-design`. **`--rescue`** redesigns an *existing* app on its untouched backend (the "client says it's too complicated" case): captures a pain inventory, re-flows the presentation, and **preserves the route/prop/field seam** (rename a prop key → blank page).
- **`/todos-sync`** — mid-work check-in: diff the project against the saved board, propose updates in batches, then run brain-capture stages A–D to surface new decisions / business rules / contradictions / scope changes from recent conversation + git.
- **`/todos-close`** — register completion: sync first, then either close the active phase or the whole project (auto-detected). Writes a phase summary or appends to `CHANGELOG.md`.
- **`/todos-week`** — plan the implementation week: interview the user, pull open tasks across every tracked project, propose a day-by-day Mon–Sun schedule, write the global `schedule.json` the Week tab reads. Cross-project — not run inside one project.
- **`/todos-dashboard`** — start the local Bun server (if not already running) and open `http://localhost:5757`.
- **`/brain-add`** — single-shot capture into `.claude/brain.db`: log a decision, business rule, contradiction, scope change, feedback item, stakeholder, or architecture note without running a full sync.

## Planning a backend autonomously (`/todos-plan`)

The usual loop interleaves planning and building one task at a time — every task stops to ask you questions before any code runs. `/todos-plan` front-loads that: it walks a **scope** of tasks (default backend-first) in **dependency order**, surfaces every ambiguity, and lets you **answer them all in one pass**. Each task ends with a **pinned contract** (method signatures, columns, routes, events) stored as a `plan` row in `brain.db`; each ambiguity is a `plan_question` row (`answer IS NULL` ⇒ the task is blocked on it). Because contracts are pinned, a task can be planned before its dependencies are built — the downstream plan relies on the fixed interface, not on guessed code. The union of all pinned contracts becomes the route/API manifest the frontend pass consumes later.

It ends with a **DAG-aware runway report**: which tasks are ready to build autonomously (grouped into context-sized phases) and which are blocked, led by the single open question that unblocks the most downstream work. Re-running is the **resume** path — it skips already-`planned` tasks and re-attempts `blocked` ones, so you extend the runway across sessions as answers arrive. Assumes a **locked scope** (PRD approved, features frozen — new asks go to the next version). Plans/questions are committed inside `brain.db` (no loose `.claude/plans/*.md` files); the runway report is a query over them, not a file scan.

Then **`/todos-build`** executes that runway. It builds the planned tasks in dependency order, each off its pinned contract in an isolated agent context, runs the backend suite after every task, and **halts the moment tests fail** — leaving the failed task open and flagging everything downstream rather than drifting. The dependency math (`ready` / `ordered` / `blocked` / transitive `downstream`) is one bun-tested module, `lib/build-order.ts`, shared by the runway report and the builder — a single source of truth, no duplicate prose. Tests are the load-bearing halt gate; the pinned-contract check is early-warning. It commits a checkpoint per phase on a `todos-build/*` branch (never the default branch, scoped `git add`), so a long unattended run stays recoverable.

When the backend is green, **`/todos-brief`** rolls every pinned contract into `.claude/docs/frontend-brief.md` — a deduped manifest of entities, endpoints (with their response shapes — *what* the UI renders, not just *where* it calls), and events, grouped by feature. That brief is the input **`/todos-design`** consumes; the frontend pass is intentionally design-forward (explore the look) rather than contract-deterministic. `/todos-design` runs the four design pillars in `lib/frontend/` — `process.md` (the workflow conductor), `knowledge.md` (visual craft), `distinctive.md` (anti-AI-slop + systems-vs-marketing), `effortless-ux.md` (usability + the audit) — to a built, dual-audited UI. The full arc: **`/todos-init` → `/todos-plan` → `/todos-build` → `/todos-brief` → `/todos-design` → `/todos-close`.**

## The Week planner

The **Week** tab (keyboard `6`) is a cross-project implementation schedule: for each day you assign which projects — and which specific tasks — you'll tackle. Scheduled tasks stay linked to their project boards by id, so completing a task anywhere updates the plan, and each day shows live `done/total` progress. Navigate weeks with `‹ prev` / `next ›` (or `[` / `]`). When a fresh week is empty and the previous week left unfinished scheduled tasks, a one-click carry-over banner offers to pull them forward. Today's slice also surfaces as a "Today's plan" section atop the Today tab.

The plan lives in **one global file**, `~/.claude/skills/todos/schedule.json` — it spans all projects, so unlike `todos.json` it is not per-project. Days are keyed by ISO date; nothing resets and past weeks stay as history. Empty days are pruned automatically. The file is gitignored (user-specific, like `config.json`).

## Project status & holding a project

A project's top-level `status` is one of `"active"`, `"shipped"`, `"archived"`, or `"on_hold"`:

- **`active`** — the normal working state. Shows in every cold/speed surface (Today, project rail, warming/stale rails, Brief, Week planner).
- **`shipped`** — set by `/todos-close` when the whole project is done.
- **`archived`** — set from the dashboard; hides the project from active views without deleting it.
- **`on_hold`** — set from the project header's **hold** button. A held project is parked for a stated reason: it drops out of the live cold/speed surfaces — Today lanes, the project rail, the warming/stale rails, the Brief, and the Week planner's *schedulable picker* (all gated on `status === "active"`) — so its coldness AND its speed/momentum freeze by exclusion, no decay-math special-casing. (A task already scheduled on a day *before* the hold stays in that day's plan as history, same as archived/shipped projects — only new scheduling is blocked.) Setting `on_hold` requires a `hold_reason` and stamps `hold_since` (ISO instant). Resuming via the header's **resume** button flips `status` back to `"active"` and clears both fields; since every write refreshes `updated_at`, a resumed project comes back **warm**, not instantly cold (fresh clock).

The **Hold** tab (keyboard `7`, placed after the Week tab) lists each held project with its name, the parking reason, "held {relative time}" from `hold_since`, and a one-click resume button. It reuses the existing stale-list / rail rendering — no parallel styling system.

## Writing `completed_at` / `created_at` / `updated_at` timestamps

**Rule:** always write the actual current UTC instant via `new Date().toISOString()` (or `date -u +%FT%TZ` from shell). Never write `YYYY-MM-DDT00:00:00Z` as a placeholder for "today" — UTC midnight converts to the **previous** local day in every timezone west of UTC, so the dashboard's Today filter (which uses local-midnight as its cutoff) will classify the task as yesterday and hide it.

When stamping multiple tasks completed in one session, **stagger the timestamps** to reflect the order they actually finished — identical timestamps collapse the Activity timeline. If reconstructing past completions from git, use the commit's UTC time (`git log --pretty=%cI <sha>` and convert the `±HH:MM` offset to `Z`).

## Data location

Per-project, all under `.claude/`:

| Path | Purpose | Commit? |
|---|---|---|
| `todos.json` | Task board (source of truth for tasks/phases) | yes |
| `brain.db` | SQLite project brain | yes |
| `brain.db-wal`, `brain.db-shm` | SQLite transient | gitignore |
| `todos.history.jsonl` | Dashboard undo log for tasks | optional |
| `todos.sync-prev.json` | Snapshot used by sync stage D | gitignore |
| `docs/`, `notes/`, `diagrams/` | Long-form content | yes |

Global, at the skill root `~/.claude/skills/todos/` (not per-project, both gitignored):

| Path | Purpose |
|---|---|
| `config.json` | List of scan roots the dashboard should watch |
| `schedule.json` | Weekly planner — which projects/tasks are scheduled on each date |

## See also

- Task schema: `lib/schema.md`
- Brain schema: `lib/brain-schema.md`
- Schedule data layer: `lib/schedule.ts`
- Reusable prompt fragments: `lib/inference-prompts.md`
- Design spec: `docs/specs/2026-04-27-todos-skill-design.md`

---
> Source: [gonoff/claude-todos-skill](https://github.com/gonoff/claude-todos-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
