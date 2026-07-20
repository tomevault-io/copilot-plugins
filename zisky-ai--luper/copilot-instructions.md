## luper

> > **For AI models** (Claude Code, Cursor, future agents) joining this codebase. Read this first to onboard fast. Cross-references are markdown links so they render both on GitHub and in Obsidian.

# AGENTS.md — Luper deep reference

> **For AI models** (Claude Code, Cursor, future agents) joining this codebase. Read this first to onboard fast. Cross-references are markdown links so they render both on GitHub and in Obsidian.
>
> Human entry point: [README.md](README.md). Binding design spec: [docs/design_spec.md](docs/design_spec.md). Permanent rationale behind the design: [docs/design_notes.md](docs/design_notes.md).

---

## 0. TL;DR

Luper = a thin Python harness driving the `claude` / `codex` CLIs through a fixed workflow:

```
brief → contract → ✋ approval gate → plan(+critic) → execute_loop(retry/replan/research) → done|partial|error
```

- **No API keys** for the executor agents — uses the local CLIs under the user's subscription.
- **OpenAI API** is used only for Deep Research (`o3-deep-research`, opt-in).
- **Single user, single task at a time, single process.** No multi-tenancy, no DB.
- **State on disk:** `tasks/<id>/state.json` + `events.jsonl` + `llm_calls.jsonl` + artifacts. No SQLite, no LangGraph.
- **Cockpit** = FastAPI + HTMX, a read + control window over `tasks/`.

Current state: **Post-MVP, Cesta A (incremental polishing path), public-release prep underway.** A0–A5 sprints + the post-test-7 batch (P0.1 frontmatter-safe regex, P0.2 monitor de-spam recipe, P1.1 critic caveats, P1.4 artifacts-only finalize) are done. The 2026-05 dual-model audit ([`docs/audit_2026-05_synthesis.md`](docs/audit_2026-05_synthesis.md)) is the basis for the post-audit cleanup PRs sequenced in the internal implementation plan (private archive). The supervisor agent (brief-launch skill) is now the default operating model (see [docs/design_notes.md "Why supervisor agent is now default"](docs/design_notes.md#why-supervisor-agent-is-now-default-post-audit-2026-05-transition)).

---

## 1. Mental model

### 1.1 What this is NOT

- Not an AI app for end users. There is no chat interface, no agent dialogue.
- Not a multi-step agent like AutoGPT — the workflow is **fixed**, not LLM-decided.
- Not a state-machine engine — it's a `while` loop with explicit phases.
- Not a paid-by-call service — uses subscription CLIs and never reports cost.

### 1.2 What it IS

A **harness** (text-defined) that orchestrates a fixed sequence:

1. **Brief** — user writes a markdown task description.
2. **Contract** — `claude` agent reads the brief and produces a structured contract (goal, deliverables, acceptance_criteria).
3. **Approval gate** — runner pauses, user reviews the contract in the cockpit, edits the markdown if needed, clicks Approve.
4. **Plan** — `claude` agent splits the contract into 1–20 steps (each step has its own acceptance_criteria). `codex` agent acts as the critic; the planner finalises.
5. **Execute loop** — for each step:
   - Executor (`claude` or `codex` per step) produces artifact(s).
   - Runner runs **deterministic checks** (word_count, regex_present, min_bytes, citation_count, python_ast_parse).
   - Verifier (`claude` agent) decides pass / fail / inconclusive based on artifacts + deterministic results.
   - On pass: critic loop (codex) reviews the verdict, verifier finalises.
   - On fail / inconclusive: retry up to `max_retry_per_step`. Then replan up to `max_replan_per_task`. Then partial.
6. **Done | Partial | Error | Stopped | Discarded** — terminal state.

The runner is **deterministic** in routing — no LLM decides "what next". Only the agents inside each phase use LLMs.

### 1.3 Workflow as text

The workflow is defined in `workflow/`:

- [workflow/workflow.md](workflow/workflow.md) — human prose description of the phase sequence.
- [workflow/prompts/](workflow/prompts/) — agent system prompts (`contract.md`, `planner.md`, `executor.md`, `verifier.md`, `critic.md`, plus shared `_system_overview.md`).
- [workflow/schemas/](workflow/schemas/) — JSON Schema for `contract.json`, `plan.json`, `verdict.json`.

Each task **snapshots** the entire `workflow/` directory into `tasks/<id>/workflow_snapshot/` at creation time. The runner reads the snapshot, not the live `workflow/`, so a task remains reproducible even if you edit prompts mid-way.

---

## 2. Repo map

```
runner/             async Python orchestrator
cockpit/             FastAPI + HTMX read+control UI
workflow/                  text + JSON: workflow.md, prompts/, schemas/
tasks/                     gitignored runtime state
tests/                     pytest, includes integration suites
docs/                      spec + findings + design notes
```

Key files (with absolute role):

| File | Role | Critical? |
|---|---|---|
| [runner/orchestrator.py](runner/orchestrator.py) | Main `while` loop. Drives the task through phases. Owns state.json. | ⭐⭐⭐ |
| [runner/phases.py](runner/phases.py) | One function per phase: `run_contract`, `run_plan`, `run_executor`, `run_verifier`, `run_research_step`. | ⭐⭐⭐ |
| [runner/cli.py](runner/cli.py) | Single point where LLM subprocess calls happen. Logs everything to `llm_calls.jsonl`. | ⭐⭐⭐ |
| [runner/sessions.py](runner/sessions.py) | Persistent claude session metadata (Sprint 6). Deterministic UUIDv5. | ⭐⭐ |
| [runner/state.py](runner/state.py) | `TaskState` pydantic model. Atomic save. State machine helpers (`mark_running`, `mark_terminal`, `mark_paused`). | ⭐⭐⭐ |
| [runner/deterministic.py](runner/deterministic.py) | Pre-LLM acceptance checks. `regex_present` has a 5 s timeout via the third-party `regex` lib. | ⭐⭐ |
| [runner/criteria_lint.py](runner/criteria_lint.py) | P0.1: warns on heading regex without `(?m)` (frontmatter-unsafe). Emits `criteria_lint_warning` event. (Candidate for move to brief-author skill per audit.) | ⭐ |
| [runner/finalize.py](runner/finalize.py) | P1.4: artifacts-only finalize — generates `artifacts/README.md`, sanitises `[[inputs/X]]` / `[[artifacts/Y]]` wikilinks, validates broken links. Auto-runs on `TaskStatus.DONE`. | ⭐⭐ |
| [runner/recovery.py](runner/recovery.py) | A1: helpers for `accept-draft`, `skip-step`, `resume --recover` (state flips + audit events). | ⭐⭐ |
| [runner/validation.py](runner/validation.py) | JSON Schema validate + heuristic JSON repair. The audit recommends collapsing heuristics to a single supervised re-prompt. | ⭐⭐ |
| [runner/deep_research.py](runner/deep_research.py) | OpenAI Responses API wrapper. Background poll, partial output recovery on rate limit. | ⭐⭐ |
| [runner/events.py](runner/events.py) | Append-only `events.jsonl` writer. EventKind enum. | ⭐⭐ |
| [runner/config.py](runner/config.py) | `config.yaml` loader with mtime-based hot reload. | ⭐ |
| [runner/cli_app.py](runner/cli_app.py) | `luper` typer-based CLI. `luper run/status/watch/approve/stop/resume/discard/log/events/list`. | ⭐ |
| [cockpit/app.py](cockpit/app.py) | FastAPI single-file app. HTMX-driven live updates via polling fragments. | ⭐⭐ |
| [config.yaml](config.yaml) | Hard limits, timeouts, CLI binary paths, default models. | ⭐⭐ |

---

## 3. Data model — files on disk

### 3.1 Per-task workspace `tasks/<id>/`

```
tasks/0042-a3f1c2/
├── state.json              # authoritative TaskState (atomic-write)
├── events.jsonl            # append-only audit log
├── llm_calls.jsonl         # full prompts + responses for every LLM call
├── brief.md                # user's input
├── inputs/                 # optional input files copied at create_task
│   └── ...
├── contract.json           # produced by run_contract
├── contract_approval.json  # written by cockpit on approve (may include edited contract)
├── contract_edited.md      # markdown draft (cockpit save-as-draft)
├── plan.json               # produced by run_plan
├── plan_draft.json         # planner's pre-critic draft
├── plan_v1.json            # archived plan after replan #1
├── critic_feedback_plan.md # critic's response to planner draft
├── executor_<step>.json    # executor's output per step (artifacts list, summary)
├── deterministic_<step>.json
├── verdicts/<step>_verdict.json
├── caveats.jsonl           # P1.1 ledger of accepted non-blocking critic caveats
├── artifacts/              # the actual deliverables — what the user gets
│   ├── README.md           # P1.4 finalize output (auto on DONE) — deliverable map + sanitised wikilinks
│   └── *.md|.py|...
├── sessions/               # Sprint 6: claude session metadata
│   ├── planner__task.json
│   ├── executor__task.json
│   ├── verifier__step_<id>.json
│   └── critic__plan.json (rare; codex critic doesn't use sessions)
├── workflow_snapshot/      # frozen copy of workflow/ at task creation
│   ├── workflow.md
│   ├── prompts/
│   └── schemas/
├── reports/summary.md      # generated on terminal status
├── stop.signal             # touch file from cockpit Stop / loop stop
└── progress.md             # human-readable snapshot, regenerated each phase
```

### 3.2 `state.json` — single source of truth

[runner/state.py](runner/state.py) defines `TaskState`. Key fields:

- `status` — TaskStatus enum: `new | awaiting_contract_approval | running | paused | stop_requested | stopped | partial | done | error | discarded`
- `pause_reason` — `quota | user | transient_error | iter_limit | time_limit | critic_unavailable | convergence_plateau`
- `current_phase` — `CONTRACT | PLAN | EXECUTE | RESEARCH | VERIFY | …`
- `iteration_count`, `replan_count`, `research_count` — checked against `config.limits`
- `steps: dict[step_id, StepRuntime]` — per-step `retry_count`, `completed`, `last_verdict_outcome`, `research_counted`
- `last_error`, `started_at`, `finished_at`, `effective_contract_path`

**Authority:** the runner owns `state.json`. The cockpit reads it; the cockpit writes only `contract_approval.json` and `stop.signal`. The CLI app does the same.

**Atomic save:** `tempfile.NamedTemporaryFile` → write → `os.replace` (POSIX-atomic).

### 3.3 `events.jsonl` — audit log

Append-only JSON-lines. Each line:

```json
{"ts": "2026-04-29T08:32:11.123456Z", "kind": "phase_started", "phase": "PLAN"}
```

EventKind values are listed in [runner/events.py](runner/events.py) (`PHASE_STARTED`, `STEP_SELECTED`, `VERDICT_FINALIZED`, `ROUTE_DECISION`, `RETRY`, `REPLAN`, `PAUSED`, `AUTO_RESUME_SCHEDULED`, …).

### 3.4 `llm_calls.jsonl` — every LLM call visible

Each line records: `phase`, `source` (`claude|codex|openai_api`), `model`, `duration_ms`, `input_tokens`, `output_tokens`, `system_prompt`, `user_prompt`, `response_text`, `error`, plus session metadata (Sprint 6: `session_id`, `is_first_turn`).

This is **the** audit trail. Per binding spec §4.3: every LLM call must be inspectable.

---

## 4. Critical flows

### 4.1 Phase: CONTRACT

[runner/phases.py:run_contract](runner/phases.py)

1. Build user prompt: brief + listing of inputs/.
2. Call claude agent with `contract.md` system prompt + `contract.schema.json` enforced.
3. **Repair loop**: if response fails JSON parse OR schema validate, send a "your last output was wrong, fix" prompt back into the **same session** (Sprint 6). Up to `_MAX_REPAIR_ATTEMPTS = 3` retries (the audit recommends reducing to 1).
4. Write `contract.json`.

Auto-fix LLM enum typos before validation: `_KNOWN_TYPOS` in [phases.py:60](runner/phases.py#L60) catches `llm_judgml → llm_judgment`, `regex_match → regex_present`, etc. Saves a repair round-trip. (The audit recommends removing this table; modern models don't reproduce the typos any more.)

### 4.2 Phase: APPROVAL gate

The orchestrator returns control to the user:

- If `contract_approval.json` doesn't exist → set status `awaiting_contract_approval`, exit `run()`.
- The cockpit shows the contract as a **markdown editor** (`contract_md.py` does the JSON ↔ markdown round-trip).
- User clicks Approve → cockpit writes `contract_approval.json` with `{approved: true, edited: bool, contract: <edited-or-null>}`, then triggers `run()` again.

If `approved=false` (Discard from the approval gate) → task moves to `DISCARDED` terminal.

### 4.3 Phase: PLAN

[runner/phases.py:run_plan](runner/phases.py)

Three claude calls (all in the **planner session**, scope `task` per Sprint 6):

1. **Draft** — planner reads contract, produces `plan_draft.json` with a placeholder `critic_decision`.
2. **Critic** — separate codex call (no session, codex doesn't support them). Reads draft, produces markdown feedback.
3. **Finalize** — planner gets critic feedback, produces `plan.json` with the real `critic_decision: {accepted, rationale, critic_summary}`.

Plan structure: 1–20 steps, each with `id, name, goal, kind: execute|research, cli, deliverable_ids, acceptance_criteria`. Research steps require a preceding execute step that produces a research prompt artifact (`produces_research_prompt_for` ↔ `consumes_research_prompt_from`). The 20-step ceiling is a hard schema limit; cost and plateau risk scale roughly linearly with step count, so the planner prompt steers toward 1–9 steps unless decomposition is genuinely required.

### 4.4 Phase: EXECUTE_LOOP

[runner/orchestrator.py:run](runner/orchestrator.py) main while-loop:

```
for each iteration:
    check stop.signal → STOPPED
    pick next incomplete step
    if no incomplete → DONE
    check hard limits (iter, wallclock, research_count)

    state.current_phase = "EXECUTE" or "RESEARCH"
    if step.kind == research:
        run_research_step  # OpenAI Responses API
    else:
        run_executor       # claude or codex per step.cli

    write executor_<step>.json
    deterministic = evaluate_step(step, artifact_paths)
    write deterministic_<step>.json

    state.current_phase = "VERIFY"
    verdict = run_verifier(...)  # claude in a per-step session
        # on pass: critic loop runs (codex), verifier finalises
    write verdicts/<step>_verdict.json

    if verdict.outcome == pass:
        runtime.completed = True
        continue
    elif retry_count < max_retry:
        retry same step (with feedback_for_retry)
    elif replan_count < max_replan:
        run replan(failure_context) → new plan, reset state.steps
    else:
        PARTIAL terminal
```

### 4.5 Sessions (Sprint 6)

[runner/sessions.py](runner/sessions.py) — claude `--session-id <uuid>` (first turn) / `--resume <uuid>` (subsequent turns). Each session is scoped per `(role, scope_key)`:

| Role | Scope | Why |
|---|---|---|
| planner | `task` | CONTRACT / PLAN / REPLAN share one conversation across the whole task |
| executor | `task` | Context across all steps + retries |
| verifier | `step_<step_id>` | Fresh per step, but stable across retries within the step |
| critic | `plan` or `verify_<step_id>` | (codex doesn't actually persist; the field is for parity) |

The session UUID is a **deterministic UUIDv5** from `(task_id, role, scope)`. So even if the metadata file is lost, the same UUID is regenerated → the claude session is recoverable.

Repair turns (when JSON parse / validate fails) reuse the **same** session. The model sees its own prior output natively in context.

**Known race** (low frequency): `get_or_create_session` writes session metadata BEFORE `call_claude` succeeds. If the CLI crashes before persisting the session, on resume the runner would `--resume` to a non-existent session. Workaround: `rm tasks/<id>/sessions/<role>__<scope>.json` and retry.

---

## 5. Hard limits and recovery

### 5.1 Limits (config.yaml)

```yaml
limits:
  max_iterations: 50           # safety net for runaway loops
  max_wallclock_seconds: 0     # 0 = disabled, we use liveness instead
  max_retry_per_step: 5
  max_replan_per_task: 1
  max_research_per_task: 3     # deep research is expensive
```

[config.yaml](config.yaml) is **hot-reloaded** (mtime cache in `runner/config.py`). Edit limits during a run, picked up at the next `load_config()` call. (The audit replaces this with `lru_cache` — restart on edit.)

### 5.2 Liveness watcher (Sprint 3)

Replaces hard wallclock abort. `_liveness_watcher` async task watches `events.jsonl` mtime. If silent past `idle_warn_seconds` (1800 = 30 min default), emits `task_idle_warning` event. If `idle_pause_seconds > 0` (default `0` = off), touches `stop.signal`.

Why: deep research can legitimately run 10–30 min, claude calls 2–5 min. Stopwatch-killing is wrong; the user should decide via cockpit. (The audit moves liveness to the supervisor skill heartbeat.)

### 5.3 Stop / pause / resume

- **Cooperative stop** — `stop.signal` touch file consumed at the top of the execute loop. Granularity: end of current phase. Used by cockpit Stop button and `luper stop`.
- **Quota auto-resume** — `QuotaExceeded` with parseable "try again in X" → `asyncio.sleep(X+5s)` and re-enter `run()`. Up to safety cap 1 h. Beyond cap → `PAUSED` with `pause_reason=quota`.
- **Recover & Resume** (Sprint 4) — for `error|partial` terminal, cockpit / CLI flips status to `paused` → `run()` picks up. The user must have addressed the root cause (edit plan.json, bump schema limit, …).

---

## 6. CLI invocation gotchas

[runner/cli.py](runner/cli.py) — single source of truth.

### 6.1 Claude

- **Never use `--bare`** — breaks OAuth subscription auth.
- **Always pass `--permission-mode bypassPermissions`** — without it, claude hangs waiting for user permission on write tools.
- `--add-dir <dir>` is **variadic** — must be followed by another non-variadic flag (`--model`, `--session-id`) before the user prompt positional arg.
- `--output-format json` — wrapper envelope `{"type":"result","result":"...","usage":{...},"is_error":bool}`. Inner `result` is the agent answer (often wrapped in ` ```json ... ``` ` markdown blocks → strip with `strip_markdown_json_block`).
- `--json-schema <inline-string>` — enforces structured output.
- **Sessions:** `--session-id <uuid>` to assign on first turn, `--resume <uuid>` on subsequent.

### 6.2 Codex

- TTY-vs-pipe mode: TTY emits `\ncodex\n` markers, pipe doesn't. [runner/cli.py:parse_codex_response](runner/cli.py#L136) detects mode by marker presence.
- Token count line `tokens used\n29 203` — Czech locale uses spaces as thousand separators. Strip non-digits.
- **No native session support** in codex. Critic remains stateless.

### 6.3 Quota detection

[runner/cli.py:_QUOTA_PATTERNS](runner/cli.py#L74) — regex patterns matching `rate limit | quota exceeded | 429 | daily limit reached | usage limit` in either stdout or stderr. Triggers `QuotaExceeded` exception regardless of exit code (some CLIs exit 0 on rate limit).

---

## 7. Cockpit (FastAPI + HTMX)

[cockpit/app.py](cockpit/app.py) — single-file app. No JS framework, no SPA.

### 7.1 Routes

- `GET /` — inbox (list of tasks, new task form)
- `POST /tasks` — create task, kick off contract phase, redirect to detail
- `GET /tasks/<id>` — detail page (status, plan, events, LLM calls, contract, artifacts)
- `GET /tasks/<id>/contract?mode=read|edit` — contract review / edit
- `POST /tasks/<id>/contract` — save contract (action=approve|draft|validate)
- `POST /tasks/<id>/{stop,resume,recover-resume,discard}` — controls
- `GET /tasks/<id>/snapshot-fragment?part=status|events` — HTMX poll fragments
- `GET /api/tasks/<id>/stream` — SSE stream of state changes (alternative to polling)

### 7.2 Live updates (Sprint 5)

`task_detail.html` has two `<div hx-trigger="every 3s">` regions:

1. `live-status` → `_status_fragment.html` (status header, pause reason, action buttons)
2. `live-events` → `_events_fragment.html` (last 12 events inline + collapsed older, last 6 LLM calls inline + collapsed older)

`<details>` use `id` + `hx-preserve` so the open/closed state survives swaps. On terminal-state transition, the fragment endpoint returns `HX-Refresh: true` so the browser does ONE full reload (surfaces the static "Output" callout panel that's only on the parent page).

### 7.3 Cockpit owns NO state

It only writes:
- `contract_approval.json` (on approve)
- `contract_edited.md` (on save-as-draft)
- `stop.signal` (on stop)

Everything else is read-only display from `tasks/<id>/`.

---

## 8. Tests

[tests/](tests/) — covers state transitions, routing, CLI wrappers, deterministic checks, validation, finalize, recovery, role overrides, criteria lint, deep research wrappers, cockpit endpoints, and a mocked end-to-end runner.

Run: `.venv/bin/python -m pytest tests/ -x -q`. Add `--no-header --tb=short` for cleaner output. The full suite hits real CLIs and takes longer; for the deterministic suite skip `test_integration_runner.py`.

---

## 9. Things that look like bugs but are intentional

- **Auto-fix walks the JSON tree replacing `type|kind|outcome|check_type` values** — looks aggressive but verified safe (no schema enum value of those keys overlaps with the `_KNOWN_TYPOS` map). The audit recommends cutting this table.
- **Czech strings in `_write_summary`, deep_research prompt prepend, and step_summary** — they live in user-facing reports / artifact-level prompts and follow `contract.language` (default `cs`). Sprint 1 ("workflow texts → English") was scoped to agent system prompts, not these.
- **Verifier sees deterministic results and is told MUST NOT change them** — by design, deterministic checks are authoritative. The verifier only adjudicates `llm_judgment` criteria.
- **`state.steps = {}` on replan** — the new plan has new step IDs; old `executor_<old_step>.json` files persist on disk for audit but aren't picked up by `_collect_artifact_summaries` (which iterates the new plan's steps). Side effect: post-replan executor doesn't see pre-replan artifacts in `previous_artifact_summaries` — the planner is expected to reference them by name in `step.goal` if needed.

---

## 10. Things that ARE actual gotchas

- **S6-1 Race** — `get_or_create_session` writes metadata before the claude CLI succeeds. If the CLI crashes before persisting the session, resume tries `--resume` to a non-existent session. Workaround: delete `tasks/<id>/sessions/<role>__<scope>.json`.
- **CC-1** — `jsonschema.RefResolver` is deprecated as of jsonschema 4.18. Will need migration to the `referencing` library eventually.
- **CC-2** — Inconsistent timestamp formats: `state.json` uses second precision, `events.jsonl` uses microsecond. The cockpit's `_local_ts` filter tolerates both.
- **CC-4** — `_run_subprocess` doesn't `try/finally: terminate(proc)` for external task cancellation. In current code, no external cancel happens to subprocess-running tasks, so it's not a live issue.
- **Stale claude session lock** — claude CLI keeps a session lock via a `.jsonl` file; after a crashed parent loop process the file plus the session-env entry remain → next invoke fails with "Session ID already in use". Manual cleanup is deterministically safe: `mv` the stale jsonl, `rmdir` the empty session-env, `luper resume --recover`.

---

## 11. Conventions when contributing

### 11.1 Current contributing rules

The project is in the public-release prep phase. The post-2026-05 audit synthesis ([`docs/audit_2026-05_synthesis.md`](docs/audit_2026-05_synthesis.md)) is the source of truth for what changes are landing and in what order. See [TODO.md](TODO.md) for the live sprint plan.

Default to **surgical fixes** during this phase. Inline fix + recovery (flip state, edit schema, mtime-reload) over "discard & restart". Big improvements → `TODO.md` backlog, not the codebase. Contract approval remains a **permanent human gate** — never auto-approve, regardless of autonomous-mode settings.

### 11.2 Code style

- Default to **no comments**. Only when WHY is non-obvious (Sprint 6 session-share rationale, regex_present timeout reason, etc.).
- Don't explain WHAT the code does — well-named identifiers do that.
- No backwards-compatibility shims, no feature flags. Just change the code.
- Atomic saves for any on-disk state (`tempfile + os.replace`).
- Pydantic v2 for typed records.
- All async — orchestrator is `async def run()`, CLI calls via `asyncio.create_subprocess_exec`.

### 11.3 Where to add what

| Adding | Where |
|---|---|
| New phase in workflow | [workflow/workflow.md](workflow/workflow.md) + new `run_<phase>` in [runner/phases.py](runner/phases.py) + branch in [runner/orchestrator.py](runner/orchestrator.py) main loop |
| New deterministic check | [runner/deterministic.py:_RUNNERS](runner/deterministic.py) + add type to schema enums + update `_KNOWN_TYPOS` if LLMs typo it |
| New criterion type | All three schemas (`contract.schema.json`, `plan.schema.json`, `verdict.schema.json`) — must be consistent |
| New agent role | [workflow/prompts/<role>.md](workflow/prompts/) + add session scope helper in [runner/sessions.py](runner/sessions.py) if it benefits from cross-call memory |
| New event kind | [runner/events.py:EventKind](runner/events.py) + emit it from the relevant phase |
| New cockpit endpoint | [cockpit/app.py](cockpit/app.py) — single file, keep it that way |
| New limit / timeout | [config.yaml](config.yaml) + corresponding pydantic field in [runner/config.py](runner/config.py) |
| New env override | [runner/config.py:_apply_env_overrides](runner/config.py) |

### 11.4 Don't

- Don't add a database. Files on disk + JSONL is the design.
- Don't add a state-machine framework. The `while` loop in `orchestrator.py:run` is intentional.
- Don't add multi-tenancy / parallel tasks.
- Don't add cost tracking / dollar reporting. Subscription is fixed; CLI errors propagate verbatim.
- Don't replace cockpit's HTMX with React. The whole point is to stay tiny.
- Don't add a settings panel. Edits go to `config.yaml`, hot-reloaded (today) or restart-required (post-audit).

---

## 12. Quick recipes

### 12.1 Run a single task end-to-end

```bash
luper run my_brief.md
luper list                              # find the task id
luper approve <task-id>                 # autonomous run starts
luper watch <task-id>                   # follow live
```

### 12.2 Inspect what an agent did

```bash
luper log <task-id> --tail 5            # last 5 LLM calls (truncated)
luper log <task-id> --full              # all calls, full prompts + responses
cat tasks/<id>/events.jsonl | jq '.kind' | sort | uniq -c
```

### 12.3 Force-flip a stuck task

```bash
# task stuck in "running" but no process actually running:
.venv/bin/python -c "
from runner.state import TaskState, TaskStatus
from runner.orchestrator import state_path
s = TaskState.load(state_path('<task-id>'))
s.status = TaskStatus.PAUSED
s.save(state_path('<task-id>'))
"
luper resume <task-id>
```

### 12.4 Recover from a broken plan.json

```bash
# Edit tasks/<id>/plan.json by hand
luper resume <task-id> --recover        # accepts error|partial state
```

### 12.5 Reset a session that `--resume` can't find

```bash
rm tasks/<task-id>/sessions/<role>__<scope>.json
luper resume <task-id>                  # next call assigns a fresh --session-id
```

### 12.5.1 Re-run finalize on a DONE task (P1.4)

```bash
luper finalize <task-id>                # idempotent: regenerates artifacts/README.md + re-sanitises wikilinks
luper finalize <task-id> --json         # machine-readable {readme_path, sanitized_files, broken_links}
```

Auto-runs on `TaskStatus.DONE` when `config.finalize.auto = true` (default). Exit 1 = broken wikilinks (soft-warning), 3 = task not found.

### 12.6 Per-task role override (executor / critic CLI swap)

A single task can swap the executor and / or critic CLI without touching the config defaults. Pass at task creation:

```bash
luper run brief.md --executor codex --critic claude
```

Both flags are optional. Valid values: `claude` or `codex`. Defaults: per-step `cli` field for the executor and `codex` for the critic. The override persists in `state.json::executor_cli` / `state.json::critic_cli` and is read at every call site (`run_executor`, `_run_critic`) so it survives `luper resume` from a fresh shell.

---

## 13. Where to look first when something breaks

| Symptom | Look at |
|---|---|
| "task stuck on PAUSED" | `tasks/<id>/state.json` `pause_reason` field, then `events.jsonl` last 20 lines |
| "schema validation keeps failing" | `tasks/<id>/llm_calls.jsonl` for the failing phase — see what the agent actually emitted |
| "executor produced wrong output" | Read the agent's full response in `llm_calls.jsonl`; check if the system prompt in `workflow_snapshot/prompts/<role>.md` is clear |
| "verifier always says fail" | Check `tasks/<id>/deterministic_<step>.json` — verifier respects deterministic results, may be them failing |
| "cockpit shows stale data" | Cockpit reads from disk every poll, no caching. Check files actually changed (`ls -la tasks/<id>/`). |
| "claude CLI hangs" | Check if `--permission-mode bypassPermissions` is in `config.yaml:cli.claude.common_args`. |
| "deep research never returns" | `events.jsonl` for `research_query_poll` heartbeats. If absent, the API call may be dead — `Stop` and resume. |
| "task ends in error during JSON parse" | Repair loop max attempts (`_MAX_REPAIR_ATTEMPTS = 3`) exhausted. Inspect `llm_calls.jsonl` for repair turns; consider adding a typo to `_KNOWN_TYPOS` if LLMs systematically hit it. |
| "artifacts/README.md missing or wrong after DONE" | `events.jsonl` for `finalize_completed` / `finalize_skipped` / `finalize_broken_links`; re-run `luper finalize <id>` (idempotent). |
| "codex executor wrote nothing AND verifier feedback names sandbox/permission" | `runner/cli.py::call_codex` must receive `sandbox="workspace-write"` + `cwd=workspace` when codex acts as executor; the critic path stays read-only. |
| "stale-session lock" | `mv tasks/<id>/sessions/<role>__<scope>.jsonl{,.stale}`, `rmdir` the empty session-env, then `luper resume --recover`. |

---

## 14. Glossary

- **Agent / role** — one of: contract, planner, executor, verifier, critic. Each has its own system prompt in `workflow/prompts/<role>.md`.
- **Phase** — one of: BRIEF, CONTRACT, CONTRACT_GATE, PLAN, EXECUTE, RESEARCH, VERIFY, ROUTE, DONE. Phases are the unit of progress in `events.jsonl`.
- **Step** — one entry in `plan.json:steps`. Has `id`, `kind: execute|research`, `acceptance_criteria`. The execute loop iterates steps.
- **Deliverable** — one entry in `contract.json:deliverables`. The user-promised output. Steps fulfill deliverables via `step.deliverable_ids`.
- **Acceptance criterion** — one entry in `step.acceptance_criteria` or `deliverable.acceptance_criteria`. Has `text`, `type`, `params`. `type=llm_judgment` → verifier decides; others → deterministic.
- **Verdict** — output of one verifier call: `outcome: pass|fail|inconclusive` plus per-criterion results, evidence, optional `feedback_for_retry`.
- **Workflow snapshot** — frozen copy of `workflow/` taken at task creation; the runner reads it instead of live `workflow/`.
- **Session** (Sprint 6) — claude `--session-id` UUID enabling persistent conversation across multiple CLI calls. Per-role, per-scope.
- **Stop signal** — touch file `tasks/<id>/stop.signal`. Cooperative stop, consumed by the orchestrator at the next loop top.
- **Critic loop** — codex agent reviews the planner draft / verifier verdict. Output is markdown feedback. The caller (planner / verifier) integrates and finalises.

---

## 15. Useful external context

- [README.md](README.md) — human entry point, setup, navigator
- [docs/design_spec.md](docs/design_spec.md) — binding spec (EN)
- [docs/design_notes.md](docs/design_notes.md) — permanent rationale behind the design
- [docs/api_reference.md](docs/api_reference.md) — stable CLI / event / state contract for recipes
- [docs/brief_author_guide.md](docs/brief_author_guide.md) — how to write a brief (humans and AI models)
- [docs/audit_2026-05_synthesis.md](docs/audit_2026-05_synthesis.md) — dual-model code audit synthesis
- [docs/cli_findings.md](docs/cli_findings.md) — claude / codex CLI operational notes
- [docs/_archive_index.md](docs/_archive_index.md) — index of files removed in PR 8 cleanup, with recovery recipe
- [TODO.md](TODO.md) — active sprint plan
- [config.yaml](config.yaml) — limits, timeouts, CLI binaries, default models

---
> Source: [Zisky-ai/Luper](https://github.com/Zisky-ai/Luper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
