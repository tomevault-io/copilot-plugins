## orca

> 🐋 Workflow-agnostic multi-pane orchestrator. Spawns and coordinates worker sessions in cmux or tmux panes via pluggable playbooks. Use when the user invokes /orca, asks to "orchestrate", "manage workstreams in parallel", "run multiple agents", or wants something to keep multiple Claude/codex/dev-server sessions advancing without manual juggling. Orca picks a backend at runtime (cmux if available, else tmux), reads playbooks from .orca/playbooks/ (project) or ~/.orca/playbooks/ (global) or its bundled defaults, and drives workers via spawn/send/read primitives. Orca NEVER writes project code — it only delegates to worker panes.


# 🐋 orca

Workflow-agnostic multi-pane orchestrator. Spawns workers in cmux or tmux panes, applies a **playbook** that defines the workflow, monitors progress, escalates when stuck.

## Core Constraint

**Orca never writes project code.** No Edit/Write/Bash on project files outside `.orca/`. If a task requires touching code, spawn a worker pane and delegate. The intelligence lives in playbooks — orca is just the muxer + dispatcher.

## On Invocation

| User says | Action |
|-----------|--------|
| `/orca` | Read `.orca/state.json` if present; otherwise enter planning mode (ask what to orchestrate) |
| `/orca <playbook> key=value …` | Apply named playbook with args (skip planning) |
| `/orca status` | Capture-pane each tracked worker, summarize signals |
| `/orca stop <pane-ref>` | Close one worker pane, update state |
| `/orca kill` | Close all orca-managed panes, clear state |
| `/orca --voice` (or `/orca --voice on`) | Enable voice I/O for the current orca pane (see [Voice mode](#voice-mode-orca---voice)). After processing the flag, continue with whatever else was on the command line — `/orca --voice` alone behaves like `/orca`; `/orca --voice gsd phase=2` applies the playbook. |
| `/orca --voice off` | Disable voice I/O (unregister the orca pane; daemon keeps running so other tools can speak). |

## Backend Detection (first thing every run)

```bash
if cmux current-window >/dev/null 2>&1; then
  ORCA_BACKEND=cmux
elif [[ -n "${TMUX:-}" ]]; then
  ORCA_BACKEND=tmux
else
  echo "🐋 orca needs cmux or tmux. Launch tmux (tmux new -s orca) or open cmux, then re-run."
  exit 1
fi
```

Save the chosen backend to `.orca/state.json` so subsequent ticks don't re-probe. See [references/backends.md](./references/backends.md) for the full primitive table — every other section in this skill assumes you've selected one.

## Playbooks

A playbook is a YAML-frontmatter markdown file telling orca how to drive a particular kind of worker (GSD phases, codex agentic loop, dev server, etc.).

**Lookup precedence** (first match wins on name conflict):
1. `./.orca/playbooks/*.md` — project-local
2. `~/.orca/playbooks/*.md` — user global
3. `~/.claude/skills/orca/playbooks/*.md` — bundled defaults

**Frontmatter shape** (full spec in [references/playbook-format.md](./references/playbook-format.md); worked example in [`playbooks/gsd.md`](./playbooks/gsd.md)):

```yaml
---
name: gsd
description: GSD multi-repo phase orchestration
triggers: ["gsd", "run phase"]
params:
  - name: repo
    required: true
  - name: phase
    required: true
supported_agents: [claude-code]            # which agents can run this playbook as workers
default_agent: claude-code
spawn:
  cwd: "{repo}"
  agents:
    claude-code:
      launcher: cdp                        # auto-perms variant per agent; see "Auto-perms" below
      initial: "/clear && /gsd:plan-phase {phase}"
    codex:
      launcher: codex --dangerously-bypass-approvals-and-sandbox
      initial: "Read .planning/STATE.md and execute phase {phase}'s plan…"
    pi:
      launcher: pi                         # pi has no permission prompts by default
      initial: "Read .planning/STATE.md and execute phase {phase}…"
watch:
  - pattern: "PHASE N COMPLETE ✓"
    action: advance
  - pattern: "Do you want to make this edit?"
    action: send_enter
stop_when:
  - "MILESTONE COMPLETE"
---
```

A playbook describes the **work** in an agent-agnostic way. Each entry under `spawn.agents` is the launcher + initial-prompt mapping for one agent runtime. `supported_agents` declares which are actually viable (e.g., GSD is Claude-Code-only because the `/gsd:*` commands are Claude Code skills). The orchestrator picks an agent per worker — defaulting to `default_agent`, overridable per-spawn.

The markdown body holds free-form notes orca reads as natural-language guidance (quirks, escalation hints, manual recovery steps).

## Agents as orchestrator OR worker

Coding agents (Claude Code, codex, pi) are interchangeable runtimes. Any of them can run **as orchestrator** (executing this skill / its peer configs) or **as worker** (spawned by an orchestrator, executing a playbook). The orca repo therefore ships multiple orchestrator configs so each agent has a way to play that role:

| Agent | Orchestrator config | Lives at |
|-------|--------------------|----------|
| Claude Code | `SKILL.md` (this file) | `~/.claude/skills/orca/SKILL.md` |
| codex | `AGENTS.md` (codex's project-config convention) | repo root, codex picks it up automatically when launched in the dir |
| pi | TBD — pi has TS Extensions / Skills / Prompt Templates; orca-as-pi-skill is a future addition | `pi/` directory in repo (placeholder) |

A worker pane only ever runs an agent in single-agent mode — the orchestration skill is irrelevant there. Workers consume the **playbook**, not orca itself.

## Auto-permission Mode

All worker launchers default to their **dangerous/auto-approval** flag. Orca cannot babysit interactive permission prompts — they would block every action.

| Tool | Auto launcher |
|------|---------------|
| Claude Code | `cdp` (skip-permissions wrapper) |
| codex | `codex --dangerously-bypass-approvals-and-sandbox` (or current full-auto flag) |
| Other agentic CLIs | their equivalent unattended flag |

A playbook may set `launcher_mode: safe` to opt out, but this is rare — the user has already accepted the safety tradeoff for orchestration ergonomics.

## Delegate perception to subagents

Orca's main context fills fast with **raw pane outputs, file dumps, log tails** — high tokens, low information. Each `cmux read-screen` lands 500–2000 tokens of mostly-noise; each `git show` on a sprawling commit is similar; multi-kilobyte worker briefs land verbatim in context when written from the main thread. Across many ticks this dominates context spend even though orca itself does coordination, not analysis.

**Fix**: route perception-heavy reads through Claude Code's `Agent` tool. The subagent has its own ~200K context window and only its **final response** returns to orca's main context. A 1500-token pane capture becomes an 80-token "still executing, last action: edit foo.ts" — ~20:1 reduction on the perception layer.

### When to delegate

- **Polling worker panes** — read the surface, return a 2-line status (signal + last action). Per-tick savings compound across many workers and many ticks.
- **Reading large state files** — `.orca/state.json` past ~5KB; `.orca/log.md` past a few sessions. Subagent extracts the specific fields/sessions you asked about.
- **Reading dev-server / pane logs** — tails are mostly HMR + access-log noise. Subagent grep-and-summarizes errors / warnings / 401–500 responses.
- **Drafting worker briefs** — multi-kilobyte briefs are wasteful in main context. Subagent writes the brief to `.orca/logs/brief-<id>.md` and returns just the file path.
- **Reading screenshots** — visual context is heavy. Subagent describes the image in 2–3 lines.
- **Inspecting big git diffs** — `git show <sha>` on a sprawling commit; subagent extracts what changed at the file/intent level.

### When NOT to delegate

- **Spawning workers** — `cmux new-split` is a single tool call returning one OK line. Round-trip cost > savings.
- **Sending text to a known pane** — single `cmux send`; cheap and the result is empty.
- **`state.json` updates** — `Edit` is bounded and the diff is small.
- **Cases where the result IS the next decision input** — e.g., "is worker_id X still active?" The subagent overhead exceeds the savings; just `Read` and decide.

### How (concrete patterns)

**Poll subagent** — replaces a direct `cmux read-screen` in Step 5:

```
Agent({
  description: "Poll worker",
  subagent_type: "general-purpose",
  prompt: "Run `cmux read-screen --surface surface:N --lines 25` and return: (1) one-line current status, (2) any errors visible, (3) whether IMPLEMENTATION_COMPLETE or IMPLEMENTATION_BLOCKED appears in the output. Don't return the full pane text."
})
```

**Brief-drafting subagent** — replaces composing a multi-kilobyte brief inline before sending it to a worker:

```
Agent({
  description: "Draft worker brief",
  prompt: "Write a brief at `.orca/logs/brief-<worker_id>.md` following references/worker-brief-template.md. Scope: <one-line>. Params: <key=value …>. Return only the absolute path of the file you wrote."
})
```

**Log-analysis subagent** — replaces tailing a noisy server log:

```
Agent({
  description: "Tail server log",
  prompt: "Read /tmp/wha-flo-dev.log lines 1-500. Return: errors (with line numbers), warnings, any 401/500 responses, and a 2-line summary. Skip routine HMR + access-log noise."
})
```

### Token math

- Direct pane poll: ~1500 tokens of raw output enter orca context.
- Subagent poll: ~80 tokens (request + 2-line return) enter orca context.
- ~20:1 reduction on the perception layer. Across ~20 ticks per session and N workers, this is the difference between "session ran out of context" and "session kept going."

### Coordination implications

- **Subagents are synchronous** — orca waits for the result before continuing the same turn. That's fine; coordination doesn't need parallelism, just lean reads.
- **Results aren't ground truth indefinitely** — re-poll between substantial state changes. Don't cache a subagent's "no errors visible" across many ticks.
- **Don't chain subagents from subagents** — flatten. Each delegation should hit a single perception target and return.

## Procedure

Follow these steps in order on every `/orca` invocation. Numbered steps are prescriptive — do them, in this order, every time.

### Step 1. Detect or read backend

```bash
if [[ -f .orca/state.json ]]; then
  BACKEND=$(jq -r '.backend' .orca/state.json)
else
  if cmux current-window >/dev/null 2>&1; then BACKEND=cmux
  elif [[ -n "${TMUX:-}" ]]; then BACKEND=tmux
  else echo "🐋 needs cmux or tmux. Launch one and re-run."; exit 1
  fi
fi
```

Persist `BACKEND` to `state.json` on first run. Never re-detect once persisted.

### Step 2. Load or initialize state

If `.orca/state.json` exists, parse it and reconcile:
- For each `worker`, capture the pane (`cmux read-screen --surface <ref> --lines 30` or `tmux capture-pane -t <ref> -p -S -30`).
- If capture fails, mark `last_signal: dead`. GC dead workers on next tick.

If absent, create `.orca/` directory and write a fresh `state.json` skeleton (workers: [], questions_pending: [], backend, started_at, cwd) — see [references/state-schema.md](./references/state-schema.md).

### Step 3. Route by user input

| User input | Route |
|------------|-------|
| `/orca` (no args) | Step 4a: planning mode |
| `/orca <name> key=value …` | Step 4b: apply playbook directly |
| `/orca status` | Step 5: report and exit |
| `/orca stop <worker_id>` | Step 6 (close one), then exit |
| `/orca kill` | Step 6 (close all), then exit |
| `/orca` arrives via `/loop` rewake | Step 5: poll + advance, then re-schedule |

**Strip flags before routing.** If `--voice` (or `--voice on` / `--voice off`) is present anywhere in the args, run the voice-mode handler from the [Voice mode](#voice-mode-orca---voice) section *first*, then strip the flag and re-evaluate the remaining args against this table (so `/orca --voice` alone routes to Step 4a; `/orca --voice gsd phase=2` routes to Step 4b).

### Step 4a. Planning mode

1. List available playbooks (scan `./.orca/playbooks/`, then `~/.orca/playbooks/`, then bundled `playbooks/`).
2. Ask the user what to orchestrate. Match their answer against playbook `triggers` if possible. Otherwise treat as freeform — offer to write a PRD to `.orca/prd-<slug>.md` and proceed without a playbook (manual `watch` rules from the user, captured into a one-off in-memory playbook).
3. Once a playbook is selected, gather any missing required `params` (prompt for each).
4. Confirm the plan back to the user (one line: "spawning {playbook} for {params} as {agent} worker"). Wait for ack before spawning.
5. Proceed to Step 4b.

### Step 4b. Apply playbook (spawn worker)

> **Ref format gotcha (cmux)**: every `--surface` / `--workspace` flag below expects the **full ref** (`surface:N`, `workspace:M`) — exactly as `cmux new-split` returns it on the `OK …` line. A bare numeric (`--surface 36`) is parsed as a *positional index*, not an ID, and fails with `Error: Surface index not found` on virtually every call. Capture the ref string verbatim and pass it through unchanged. Wrong: `cmux send --surface 36 …`. Right: `cmux send --surface surface:36 …`. (See [references/backends.md](./references/backends.md#primitive-table) for the full callout.)

For each worker the playbook implies (usually one — but cross-repo playbooks may declare more):

1. Resolve params (user input + playbook defaults). If any required param is missing, error and exit.
2. Pick agent: explicit `agent=` arg → playbook's `default_agent` → first `supported_agents` entry. Verify the agent block exists in `spawn.agents.<name>`.
3. Generate a `worker_id` (`{playbook}-{cwd-slug}-{disambiguator}`).
4. Spawn the pane:
   - **cmux**: `cmux new-split <direction>` (default `right`); capture `surface:N` and `workspace:M` from stdout (`OK surface:N workspace:M`). **Note**: `cmux new-split` does NOT honor `spawn.cwd` — the new pane inherits the orchestrator's cwd. If the playbook sets a different `spawn.cwd`, you must `cd` into it as part of the launcher (see step 6).
   - **tmux**: `tmux new-window -d -n "<worker_id>" -c "<spawn.cwd>"`. tmux honors `-c` directly, no extra cd needed.
5. Sleep `spawn.agents.<agent>.initial_wait_s || 5` seconds.
6. **Send the launcher**, with cwd compensation for cmux:
   - **cmux + spawn.cwd != orchestrator's cwd**: send `cd '<spawn.cwd>' && <launcher>` (single quotes around the path; assumes no embedded single quotes — if any, escape per shell rules).
   - **cmux + spawn.cwd == orchestrator's cwd**: send `<launcher>` directly.
   - **tmux**: send `<launcher>` directly (cwd was set at window creation).

   The launcher must be the auto-perms variant (`cdp`, `codex --dangerously-bypass-approvals-and-sandbox`, `pi`, etc.). Submit with `cmux send-key … enter` or `tmux send-keys … Enter`.
7. Sleep ~8s (Claude Code / codex boot time).
8. **Handle first-run trust prompt**. Both Claude Code and codex show a trust dialog the first time an agent boots in an unfamiliar directory. Capture the pane and send a single Enter (which selects the highlighted "trust / continue" option) if you see any of:
   - Claude Code: `Is this a project you trust?` / `Yes, I trust this folder`
   - codex: `Do you trust the contents of this directory?` / `Yes, continue`
   - pi: no equivalent prompt — skip

   After sending Enter, sleep 2s. If no prompt was shown, the directory is already trusted and Enter on an empty input line is harmless.
9. Send the playbook's `spawn.agents.<agent>.initial` (with param substitution applied) + Enter.

   > **Prefer subagent brief-drafting.** When the `initial` payload is a substantial brief (say, more than ~50 lines), delegate the drafting to an `Agent` subagent that writes the brief to `.orca/logs/brief-<worker_id>.md` and returns just the path — then send that file's contents (or a `cat <path>` reference) to the worker. Keeps the kilobytes of brief text out of orca's main context. See "Delegate perception to subagents" above.

10. Append the worker to `state.json` with `last_signal: spawning`. Persist:
    - `playbook` — the playbook's `name` field
    - `category` — the playbook's `category` field (default `implementation` if unset)
    - `cwd` — the resolved `spawn.cwd` after parameter substitution (canonical worktree identifier; used by Step 6 review-chain matching regardless of which param name the playbook used: `repo`, `worktree`, `target`, etc.)

    Step 6 reads all three when deciding whether to offer the review chain and how to glob review files.

### Step 5. Poll + advance (every active tick)

> **Prefer subagent polling.** For each worker, instead of `cmux read-screen` directly from orca's main thread, delegate to an `Agent` subagent that runs the capture and returns just the status signals (1–3 lines). Raw pane text never enters orca's context. See "Delegate perception to subagents" above.

For each worker in `state.json` whose `last_signal` is not `task_complete` or `dead`:

1. Capture pane (last `LINES` lines, default 30): `cmux read-screen --surface <ref> --lines 30` or `tmux capture-pane -t <ref> -p -S -30`. If capture fails twice in a row, mark `dead` and skip.
2. **Check `stop_when[]` first** (priority over `watch[]`). For each regex in the playbook's `stop_when` list: if it matches the captured text, mark the worker `last_signal: task_complete` and skip directly to step 6 (history append) — do NOT run watch[] matching or generic detection (terminal conditions trump in-flight actions). If `stop_when` is absent or no patterns match, continue.
3. Match each `watch[].pattern` against the captured text **in order**. First match wins.
4. On first match, perform the corresponding `action`:

   | Action | Implementation |
   |--------|---------------|
   | `advance` | Send `/clear`, sleep 2s, send substituted `next` + Enter. Set `last_signal: phase_complete`. |
   | `send_enter` | Send Enter only. `last_signal: waiting_input`. |
   | `send_text` | Send substituted `next` + Enter. |
   | `clear_and_send` | Send `/clear`, sleep 2s, send `next` + Enter. |
   | `stop` | Mark `last_signal: task_complete`. Pane will be closed in Step 6. |
   | `escalate` | Capture last 100 lines to `.orca/logs/<worker_id>-<ts>.txt`, add to `questions_pending`, ping user, no further action this tick. |
   | `wait` | No-op. Update `last_signal` only. |
   | `delegate` | See `## Delegation` for the full per-tick procedure. Behavior depends on parent's `awaiting`/`relay_state`: spawn child on first match, no-op while child runs, replay queued responses keyed by `{capture[0]}` once child closes. |

   Full action enum and `next` template syntax: [references/playbook-format.md](./references/playbook-format.md#action-vocabulary).

   **If the closed worker has `delegate_parent` set in state.json**, after running its own Step 6 close, also: read its review file's `delegate_relay_section` (per the parent's stored value), parse `<N>. <text>` lines into a map, and write that map to the parent's `relay_queue`. Set parent's `relay_state: "queued"`, clear parent's `awaiting`. The relay itself happens on the parent's NEXT tick when its watch rule matches again.

5. If no `watch` pattern matched, run `scripts/detect-state.sh --stdin` against the captured text for a generic signal (`executing` / `idle` / `error` / etc.). Log but don't act on generic signals — the playbook's rules are authoritative.
6. Append a `history` entry to the worker (`{ts, signal, action}`).
7. Update `state.json` atomically (`.tmp` + `mv`).

After processing all workers:
- If any worker had `task_complete` action → proceed to Step 6 for those.
- If all workers are `task_complete` or `dead` → print summary, exit.
- If user invoked `/orca status` → print one line per worker (id, signal, last-poll age) and exit.
- Otherwise → schedule next tick:
  - In `/loop` mode: `ScheduleWakeup` with delay = playbook's `poll_interval_s` (default 90s active, 1200–1800s if every worker is `idle`).
  - Outside `/loop`: print "next poll in {N}s" and exit. User re-invokes manually or via cron.

### Step 6. Close worker(s)

**Two-phase ordering** when multiple workers are queued (e.g. several review-class workers all hit `REVIEW_DONE` in the same tick):

- **Phase 1 (per-worker)**: substeps 1–4 — capture pane, close pane, remove from state if appropriate, offer review chain. Run for every queued worker.
- **Phase 2 (per-cwd, ONCE)**: substep 5 — aggregate review chain, then batch-remove. Runs only after Phase 1 has finished for every worker in the queue.

This ordering matters because aggregation in substep 5 may batch-remove sibling review workers that haven't yet had their pane captured/closed. Always close panes first, aggregate last.

For each worker to close (Phase 1):

1. Capture last 100 lines to `.orca/logs/<worker_id>-<ts>.txt` (preserve evidence).
2. Close the pane:
   - **cmux**: `cmux close-surface --surface <ref>` (add `--window <window>` if the worker is in a separate workspace).
   - **tmux**: `tmux kill-window -t <worker_id>`.
3. Remove the worker entry from `state.json.workers` — UNLESS the worker has `category: review`. **Review-class workers stay in state with `last_signal: task_complete`** until Phase 2's batch-removal. This preserves their `spawned_at` for the aggregation marker so artifacts from earlier-closed siblings aren't missed. (Mark `last_signal: dead` if recovery is plausible for non-review workers — playbook-specific.)
4. **Trigger review chain** (only when `last_signal: task_complete` AND closed worker's `category == "implementation"`):
   - Use the closed worker's `cwd` (persisted at spawn — see Step 4b#11) as the canonical worktree identifier. This works regardless of which param name the playbook used (`repo`, `worktree`, `target`, …).
   - Check whether any review-class workers (`category: review`) with the same `cwd` are already in `state.json.workers[]` or have written to `.orca/reviews/` since this worker's `spawned_at`. If yes, skip — review chain is already in flight or done.
   - **If the closed playbook declares a `chain:` field** (see `references/playbook-format.md` `## Chain`), auto-spawn each listed playbook with NO user prompt. For each chained playbook:
     - Pass the closed worker's `cwd` as the chained playbook's `worktree` (or first dir-pointing param it declares — `worktree` / `repo` / `cwd`).
     - Match other declared params by name from the parent's invocation params (e.g., parent's `app_url` → chained's `app_url`).
     - If a chained playbook declares a required param that can't be resolved, escalate to user and do NOT half-spawn — atomic chain, all-or-nothing.
     - Spawn each through Step 4b. Multiple chained playbooks spawn in the same tick (parallel).
   - **Otherwise** (no `chain:` field), surface the manual offer to the user with concrete invocations:

     > "{playbook} closed cleanly. Want me to spawn validators against {cwd}?"
     > `/orca code-review worktree={cwd} target=<PR# or commit-range>`
     > `/orca ui-validator worktree={cwd} app_url=<url>`

     Wait for confirmation before spawning. See `## Review chain` section below for the full pattern (parallel spawn, frontmatter aggregation, actionable-items handoff).
After Phase 1 finishes for every worker in the close queue, do Phase 2 once per affected `cwd`:

5. **Aggregate review chain** (only when at least one closed worker had `category == "review"`):
   - Look for sibling review-class workers (same `cwd`) in `state.json.workers[]`. **Closed-but-not-removed siblings (per substep 3) count as still-present.** If any have `last_signal != task_complete`, defer aggregation — wait for them to close on a future tick.
   - When all review-class workers for the same `cwd` are `task_complete`, run the aggregation procedure from `## Review chain`. The aggregation snippet reads `spawned_at` from the (still-present) review workers in state.json, so all sibling timestamps are available — no stashing needed.
   - After aggregation completes, **batch-remove** every `task_complete` review-class worker for that `cwd` from `state.json.workers[]`. By Phase 1's contract their panes are already closed, so removal is safe.
6. If `state.json.workers` is empty and `/orca kill` was invoked, optionally archive `state.json` to `.orca/state.<ts>.json.bak` and start fresh on next invocation.

## Polling cadence

- Active mode (any worker not `task_complete`/`dead`): **60–120s** between ticks. Default `poll_interval_s: 90`.
- Idle mode (every worker `idle` for ≥ `idle_threshold_s`): **1200–1800s**. The 5-min cache window matters here — never pick 300–600s, that's worst-of-both.
- Use `ScheduleWakeup` from `/loop` dynamic mode for self-pacing across context boundaries.

## State Files

`.orca/` lives in cwd where `/orca` was invoked. Schemas in [references/state-schema.md](./references/state-schema.md).

```
.orca/
├── state.json           # active workers, backend, last-poll timestamps, params
├── log.md               # append-only chronological session log (cross-session memory)
├── prd-{name}.md        # per-workstream requirements, used to answer worker questions
├── playbooks/           # project-local playbooks (override bundled/global)
├── reviews/             # review-class playbook output (code-review, ui-validator, …)
└── logs/{pane}-{ts}.txt # captured pane output on escalation
```

### Session log (`.orca/log.md`)

`state.json` only holds the *current* state — once a worker is removed, only its `completed_workers` entry survives, and that's terse. The orchestrator's reasoning, decisions, friction patterns, and cross-worker context are lost across sessions unless captured.

Maintain `.orca/log.md` as an **append-only** human-readable timeline. On every session, the orchestrator should:

1. **First invocation of a session** — append a `## YYYY-MM-DD — <one-line session theme>` header.
2. **On every meaningful event** — append a row or bullet under the current session:
   - Worker spawned: `| HH:MM | spawn | <worker_id> | <surface> | <one-line scope> |`
   - Worker closed: include outcome + commits landed
   - Decision made (e.g. "skipped chain", "split into N parallel", "committed worker's uncommitted diff")
   - Friction noted (e.g. "worker forgot to commit", "test endpoint kept 500-ing")
3. **At session end / handoff** — write a "Net result" paragraph naming what shipped and what's open.

Why log instead of just relying on `state.json`:
- `state.json.completed_workers` is a flat list of facts; `log.md` captures *why* and *what changed in approach*.
- A new orchestrator session reading `.orca/log.md` can pick up exactly where the previous one left off, including friction patterns to avoid.
- The user can audit a session post-hoc without scrolling pane logs.

Format guidance: keep entries terse (1 line each where possible). A table per session is fine when there are >3 workers. Include git SHAs for every shipped commit.

The log survives `/orca kill` — it is NEVER cleared. Only the user manually removes it.

## Review chain

A **review-class playbook** is one whose output is a structured findings file, not a code change. Bundled examples: `code-review`, `ui-validator`. Future: `security-review`, `perf-review`, `accessibility-review`. They all follow the same convention — see [references/review-format.md](./references/review-format.md).

### When to invoke

After an **implementation playbook** (anything that ends in `task_complete` because real work shipped — `gsd`, custom feature playbooks, …) closes its workers, the orchestrator triggers the review chain. Two paths:

- **Declarative** (preferred for known-thorough flows): the implementation playbook declares a `chain:` list in its frontmatter (see `references/playbook-format.md` `## Chain`). Orca auto-spawns those playbooks on `task_complete` — no user prompt. This is how `gsd-verify.md` differs from `gsd.md`: same body, but `gsd-verify` declares `chain: [code-review, ui-validator]`.
- **Manual** (default for playbooks without `chain:`): orca offers the spawn:

  > "{playbook} finished. Want me to spawn code-review and ui-validator against {worktree}?"

  Validators consume codex/Claude budget and a few minutes of wall time. For unannotated playbooks, default to opt-in so the user picks when the cost is worth it.

### Spawning multiple validators in parallel

review-class playbooks don't depend on each other. Spawn them in the same tick:

```bash
# Each invocation goes through Step 4b independently
/orca code-review worktree=<path> target=<PR# or commit-range> [prd=<path>]
/orca ui-validator worktree=<path> app_url=<url> [target=<feature>] [acceptance_criteria=<path-or-text>]
```

Both end up in `state.json.workers[]` as separate entries. They run side-by-side in their own panes, write to `.orca/reviews/`, and close themselves via their `stop_when` rules.

### Aggregation (after validators close)

Aggregation runs from **Step 6 Phase 2** — after Phase 1 has captured and closed every queued worker's pane for the current tick. Do NOT aggregate during an individual worker's Phase 1 close: that recreates the same-tick race where a sibling's state entry vanishes before its pane is captured. The trigger is "Phase 1 for this tick is finished AND at least one closed worker had `category: review`," not "a single review worker just closed."

1. Find all review files written since the earliest review worker's `spawned_at`. Review-class workers are kept in state.json with `last_signal: task_complete` until this aggregation runs (see Step 6 substep 3) — so the closing worker AND any earlier-closed siblings are all readable here.

   ```bash
   # All task_complete review workers for this cwd are still in state. Find the earliest spawned_at.
   earliest_spawned_at=$(jq -r --arg cwd "$CWD" \
     '.workers[] | select(.category=="review" and .cwd==$cwd) | .spawned_at' \
     .orca/state.json | sort | head -1)

   # Convert ISO8601 UTC → epoch seconds (portable across BSD/GNU date)
   if epoch=$(date -j -u -f '%Y-%m-%dT%H:%M:%SZ' "$earliest_spawned_at" +%s 2>/dev/null); then : ;  # macOS BSD date
   else epoch=$(date -u -d "$earliest_spawned_at" +%s); fi                                            # GNU date

   # Create a marker file with that mtime, glob via -newer (POSIX, no date-parse needed in find)
   marker=$(mktemp)
   if [[ "$(uname)" == "Darwin" ]]; then
     touch -t "$(date -r "$epoch" +%Y%m%d%H%M.%S)" "$marker"     # BSD touch
   else
     touch -d "@$epoch" "$marker"                                # GNU touch
   fi
   find .orca/reviews -type f -name "*.md" -newer "$marker"
   rm "$marker"
   ```

   Why a marker file instead of `find -newermt "$ts"`: BSD `find` rejects ISO8601 with `Z` ("Can't parse date/time"), and BSD vs GNU silently disagree on whether `T`-without-`Z` is UTC or local. Marker mtime is unambiguous on both platforms.
2. For each file, read **frontmatter only** (skip the body for triage):
   ```bash
   awk '/^---$/{c++; next} c==1{print} c==2{exit}' <file>
   ```
   Capture: `type`, `status`, `issue_count`, `target`.
3. Build the user-facing summary:
   ```
   Review chain results:
   - code-review: <status> (<issue_count> issues) — target <target>
   - ui-validation: <status> (<issue_count> issues) — target <target>
   ```
4. If any `status` is `fail` or `needs-attention`, read those files' **`## Actionable items for next agent`** sections (the orchestrator's primary handoff target) and present them to the user with options:
   - **Forward to a fix run**: `/orca gsd repo=<...> phase=fix` (or whatever implementation playbook is appropriate), passing the actionable items as the brief.
   - **Send to specific live worker panes**: if any other workers are still in `state.json.workers[]`, `cmux send --surface <ref>` the relevant items per pane. (Manual in v1 — no `dispatch_to <worker>` action exists yet.)
   - **Stop here**: user reads, decides what to do offline.
5. If every `status` is `pass`, report clean and stop.

### When to stop iterating

A "fix → re-validate → fix again" loop is useful but can run forever finding smaller and smaller nits. Stop the chain when ANY of these are true:

1. **Run reports `status: pass`** — no actionable items. Ship.
2. **Two consecutive runs find only `medium` / `low` issues** (zero `high` / `critical` across both runs). The pattern is doc nits, not functional bugs. Stop, add the remaining items to a project backlog, ship.
3. **Three iterations on the same target** — hard cap. Beyond this, diminishing returns dominate; the cost of another full validator pass exceeds the marginal value. Triage what's left manually.

When stopping for reasons 2 or 3, surface the remaining items to the user explicitly (e.g. "stopping iteration — 2 medium-severity items not addressed: [list]. Add to backlog or ship as-is?"). Never silently drop findings.

The validator briefs already tell codex/claude not to manufacture nits ("if the diff is genuinely clean, `status: pass` and `issue_count: 0`"), but the orchestrator is the final gate — even a strict reviewer can't avoid finding *something* worth mentioning on every pass.

### What NOT to do

- **Don't** rewrite review files. They're append-only artifacts of the review run.
- **Don't** auto-chain into a fix playbook without confirmation. Findings could be wrong, the user should triage.
- **Don't** delete `.orca/reviews/` on `/orca kill`. They survive the session — `kill` only closes panes and clears `state.json.workers[]`.
- **Don't** treat a missing review file as success. If a validator closed without writing one, that's a `dead`/`error` signal, not pass.
- **Don't** keep iterating past the convergence rules above just because the validator is willing to. Token budget and wall time matter.

## Delegation

The `delegate` watch action lets a parent worker offload a question/checkpoint to a child playbook *mid-flow*, then relay the child's answer back into the parent's pane via `send_text`. Use case: GSD's `/gsd:verify-work` runs a series of `CHECKPOINT: Verification Required` prompts; orca delegates each to ui-validator, which exercises the test in a real browser and writes structured responses. Parent never blocks on a human.

See `references/playbook-format.md` `## Delegate action` for syntax.

### Lifecycle

A worker spawned by a `delegate`-action match is called a **delegate child**. It's a normal playbook spawn (Step 4b) but tagged with `delegate_parent: <parent_worker_id>` in `state.json` so orca can route results back later.

State fields added on the parent at first match:

```json
{
  "worker_id": "gsd-uat-01",
  "awaiting": "ui-val-02",          // the delegate child's worker_id
  "delegate_relay_section": "## UAT Responses",  // copied from watch rule
  "relay_queue": null,              // populated when child closes
  "relay_state": "awaiting"         // "awaiting" | "queued" | "drained"
}
```

### Procedure (per tick)

When evaluating watch rules for a worker:

1. **First match of a `delegate` rule** (parent's `awaiting` is null):
   - Substitute `delegate.with` params (incl. `{capture[N]}`, `{cwd}`, etc.).
   - Spawn the child via Step 4b. Child's `delegate_parent` in state.json points back to this worker.
   - Set parent's `awaiting: <child_worker_id>`, `delegate_relay_section`, `relay_state: "awaiting"`.
   - Do NOT send anything to parent's pane this tick.
   - Skip remaining watch rules for the parent this tick.

2. **Match while parent's `awaiting` is set AND `relay_state == "awaiting"`** (child still running):
   - No-op for this rule. Don't re-spawn, don't relay.
   - Other watch rules that don't conflict (e.g., `send_enter` for editor prompts) MAY still fire — order in `watch:` decides.

3. **Child closes** (handled in Step 6 close logic, not here):
   - Read child's review file at `.orca/reviews/<child_review>.md`.
   - Parse the `delegate_relay_section` from the body. Each line in the format `<N>. <text>` becomes a queue entry keyed by `<N>`.
   - Set parent's `relay_queue` to the parsed map (`{1: "pass", 2: "issue: ...", ...}`).
   - Set `relay_state: "queued"`, clear `awaiting`.
   - Child's worker entry follows normal Step 6 close (review-class workers stay until aggregation; non-review children are removed).

4. **Match while `relay_state == "queued"` and queue non-empty**:
   - Use the rule's pattern's first capture group (`{capture[0]}`) as the lookup key.
   - If `relay_queue[key]` exists: `send_text` that response to the parent's pane. Remove that key from the queue.
   - If queue becomes empty: `relay_state: "drained"`.
   - If key NOT in queue: log to `.orca/logs/<parent_id>-relay-miss.txt`, `escalate` to user (the validator missed a test).

5. **Match while `relay_state == "drained"`**: fall through to next watch rule. The delegate is exhausted — user can re-trigger by manually invoking the same action or letting another rule handle the prompt.

### Idle suppression

While `awaiting != null`, the parent's idle threshold is suspended. Don't mark `last_signal: idle` or escalate; the child is doing real work and the parent's pane intentionally shows no progress.

Once `awaiting` clears (relay_state queued or drained), normal idle rules resume.

### Convention for child playbooks

Children invoked via `delegate` MUST write a `<relay_section>` (default `## Responses`) to their review file with one line per response in the format `<N>. <text>`. Anything else is informational (the rest of the review still gets aggregated).

ui-validator's UAT mode (criteria points to a `*-UAT.md`) is the canonical example — see `playbooks/ui-validator.md` `## UAT mode`.

### Limitations (v1)

- One delegate child at a time per parent. A second `delegate` match while `awaiting` is set is a no-op.
- No timeout on `awaiting` — if child hangs, parent stays paused indefinitely. Manual `/orca kill <child_id>` clears it (orca should detect orphaned `awaiting` on next tick and unset).
- Queue is exhausted in match-by-key order, not insertion order. Multiple matches with the same `{capture[0]}` would only fire once per key.
- Children of children (delegate chains) are not supported. A delegate child's playbook MUST NOT itself contain `delegate` watch rules.

## Helper Scripts

```bash
~/.claude/skills/orca/scripts/poll.sh              # capture every tracked pane, classify
~/.claude/skills/orca/scripts/detect-state.sh PANE # classify one pane against playbook patterns
```

These call the backend-appropriate primitive based on `.orca/state.json`'s `backend` field.

## References

| File | Use |
|------|-----|
| [references/backends.md](./references/backends.md) | cmux + tmux primitive table, gotchas, side-by-side commands |
| [references/state-schema.md](./references/state-schema.md) | `.orca/state.json` shape, PRD template, log format |
| [references/playbook-format.md](./references/playbook-format.md) | Full YAML schema, action vocabulary, parameter substitution |

## Helper scripts

```bash
~/.claude/skills/orca/scripts/poll.sh                # capture + classify every tracked worker
~/.claude/skills/orca/scripts/detect-state.sh REF    # classify one pane via backend
~/.claude/skills/orca/scripts/detect-state.sh --stdin < pane-text  # classify without re-capturing
```

These dispatch by reading `backend` from `.orca/state.json`. Useful for one-off diagnostics outside the main loop.

## Anti-Patterns

| Don't | Do |
|-------|-----|
| Write project code from orca | Spawn a worker, delegate |
| Use `claude` instead of `cdp` (or any safe-mode equivalent) | Auto-perms launcher every time |
| Hardcode workflow logic into orca | Put it in a playbook |
| Hardcode tmux or cmux commands | Use the backend abstraction in `references/backends.md` |
| Poll faster than 60s in active mode | Sub-60s burns compute for no benefit |
| Forget to capture `cmux new-split` stdout | The new `surface:N` is in the OK line — parse it; don't re-list |
| Forget `--window` for cross-window cmux workspaces | Splits don't need it; separate workspaces do |
| Auto-close panes you didn't spawn | Only kill panes tracked in `.orca/state.json` |

## Voice mode (`/orca --voice`)

Optional. Wraps the orca pane with bidirectional voice I/O so you can listen to orca's responses and reply by talking. Workers stay text-only — voice only speaks/types **for the orca pane itself**, since orca already narrates whatever its workers are doing.

**Architecture in one paragraph.** The talk-to-claude project at `~/Code/talk-to-claude/` ships a small HTTP daemon that owns TTS, recording, transcription, and key injection. A user-global Claude Code Stop hook (`~/.claude/settings.json`) POSTs to that daemon on every Claude turn; the daemon checks whether the firing session's `$CMUX_SURFACE_ID` matches the orca pane that registered itself, and only speaks for matches. A Raycast hotkey POSTs to `/talk` to record a turn (sox VAD → Whisper → `cmux send` into the orca pane).

### Requirements

| Need | Why |
|------|-----|
| `OPENAI_API_KEY` env var | Whisper transcription |
| `sox`, `curl`, `python3` | Recording, HTTP, JSON |
| `~/.local/bin/say` (kokoro wrapper) **or** override `TTC_SAY` | TTS playback |
| Raycast Script Command directory pointing at `~/Code/talk-to-claude/raycast-scripts/` | Hotkey for push-to-talk |
| Stop hook installed in `~/.claude/settings.json` (one-time) | Forwards turn-end events to the daemon |

### Handler procedure (when `--voice` appears in args)

1. **Verify cmux** — voice mode requires `$CMUX_SURFACE_ID` to be set. If empty (e.g., running under tmux or bare terminal), abort with: "🐋 voice mode requires cmux." Do NOT proceed to start the daemon.
2. **Branch on subcommand**:
   - `--voice off` → `curl -sf -X POST http://127.0.0.1:8848/unregister` ; report "🐋 voice off (daemon kept running)" ; strip flag and continue routing.
   - `--voice` or `--voice on` (default) → continue to step 3.
3. **Start the daemon if not running** —
   ```bash
   if ! curl -sf --max-time 0.5 http://127.0.0.1:8848/health >/dev/null 2>&1; then
     # IMPORTANT: do NOT nohup/disown. cmux's socket auth uses the caller's
     # process ancestry — if the daemon detaches and reparents to launchd,
     # `cmux send` from inside the daemon fails silently (rc != 0, empty
     # stderr). Keep it as a backgrounded child of this shell so the cmux
     # ancestor chain is preserved.
     mkdir -p /tmp/talk-to-claude
     python3 "$HOME/Code/talk-to-claude/voice-daemon.py" \
       >>/tmp/talk-to-claude/daemon.stdout 2>&1 &
     for _ in 1 2 3 4 5 6 7 8; do
       sleep 0.25
       curl -sf --max-time 0.5 http://127.0.0.1:8848/health >/dev/null 2>&1 && break
     done
   fi
   ```
4. **Register the orca pane** —
   ```bash
   curl -sf -X POST http://127.0.0.1:8848/register \
     -H "Content-Type: application/json" \
     -d "{\"surface_id\":\"$CMUX_SURFACE_ID\",\"workspace_id\":\"$CMUX_WORKSPACE_ID\"}"
   ```
5. **Confirm to the user** with a one-liner that includes the registered surface and the hotkey hint, e.g.:
   "🎙 voice on (surface:NN). Press your Raycast hotkey to talk; orca's replies will be spoken."
6. **Strip `--voice` from args** and continue routing per [Step 3](#step-3-route-by-user-input). `/orca --voice` alone falls through to planning (Step 4a); `/orca --voice gsd phase=2` continues to Step 4b.

### Daemon endpoints (for diagnostics)

| Endpoint | Use |
|----------|-----|
| `GET /health` | "ok" if up |
| `GET /status` | JSON snapshot — registered surface, last assistant text, etc. |
| `POST /register` `{surface_id, workspace_id}` | Claim TTS/PTT for that pane |
| `POST /unregister` | Release the pane |
| `POST /stop` | Internal — wired from Claude Code Stop hook |
| `POST /talk` | Internal — wired from Raycast PTT hotkey |
| `POST /silence` | Kill in-flight TTS |

### Gotchas

- The daemon survives orca exits on purpose (so `/orca --voice off` then `/orca --voice` later is cheap). To fully kill it: `pkill -f voice-daemon.py`.
- The Stop hook is global, so any Claude session can register itself in principle — but only the registered surface will be served. Workers' Stop hooks no-op silently.
- Whisper occasionally hallucinates "you" / "thank you" on near-silence; the daemon already filters those.
- `cmux send` types literal characters and submits with Enter. If you want to dictate something multi-line, do it in two presses (orca's prompt buffer accepts the second message as a continuation).

---
> Source: [webdevike/orca](https://github.com/webdevike/orca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
