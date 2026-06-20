## agile-loop

> Run the queued GSD → Superpowers → review → ship loop end-to-end. Reads tasks from `docs/agile-loop/tasks/*.md`, spawns isolated child sessions per stage, opens one PR per task, and auto-merges it (squash) before continuing to the next task. Use when asked to "run the agile loop", "process the next queued task", or "drive the autonomous engineering loop".


# Agile Loop — Claude Code adapter

You are the Claude Code adapter for Agile Loop. The Codex adapter lives at `scripts/agile-loop.sh` and runs the same contract via `codex exec --ephemeral`; you run it via `claude -p` headless child sessions. Both adapters read the same `docs/agile-loop/tasks/*.md` queue and write the same `.agile-loop/status.json` (the dashboard at `scripts/agile-dashboard.py` works on either path). `references/prompts.md` is shared documentation of the host-neutral prompt shapes; the templates Claude actually sends are inlined in **Prompt templates** below so there is no runtime file lookup against the target repo.

When you spawn a child via `claude -p "<prompt>"`, that is the Claude-side equivalent of `codex exec --ephemeral`: no chat history carryover, child reconstructs all context from the repository, task file, and explicit output files.

## Pre-flight

1. Parse args (all optional): `--repo PATH` (default `$PWD`), `--base BRANCH` (default `main`), `--max-iterations N` (default 10), `--unsafe-bypass-approvals` (boolean), `--dry-run` (boolean), `--dashboard-host HOST` (default `127.0.0.1`, env `AGILE_LOOP_DASHBOARD_HOST`), `--dashboard-port N` (default `8765`, env `AGILE_LOOP_DASHBOARD_PORT`), `--no-dashboard` (boolean, env `AGILE_LOOP_NO_DASHBOARD=1`). `cd` into the resolved repo root and remember it as the absolute `REPO`.
2. Resolve approval bypass: live runs require either `--unsafe-bypass-approvals` or `AGILE_LOOP_UNSAFE_BYPASS=1` in the environment. If neither is set and `--dry-run` is also not set, stop with a clear blocked message — the Codex adapter has the same gate (`scripts/agile-loop.sh`).
3. Confirm `git`, `gh`, `claude`, and `python3` are on `PATH`. If any are missing, write a blocked status with a clear message and stop.
4. Generate a `RUN_ID` (UTC timestamp + short random). Create `.agile-loop/` and `.agile-loop/runs/<RUN_ID>/` if missing. Remember these absolute paths for the whole run: `RUN_DIR=$REPO/.agile-loop/runs/$RUN_ID` and `WORKTREES=$REPO/.claude/worktrees`. **`.agile-loop/`, `.agile-loop/status.json`, and `docs/agile-loop/tasks/*.md` always live in the main repo** (never inside a worktree) so the dashboard keeps reading them and they survive worktree teardown. Each claimed task runs in its own Claude worktree under `WORKTREES` (see **Per-iteration loop → 1. Pick the next task**). `.claude/worktrees/` lives inside the repo, but git treats a registered worktree path as a nested worktree — it does not appear as untracked in the main repo — so no `.gitignore` change is needed.
5. Initialize `.agile-loop/status.json` with `status: starting` via the status writer (see **Status writer** below).
6. **Auto-start the dashboard** so `http://<dashboard-host>:<dashboard-port>` (default `http://127.0.0.1:8765`) always reflects the active loop. See **Dashboard auto-start** below for the exact behavior. Skip this step entirely when `--no-dashboard` / `AGILE_LOOP_NO_DASHBOARD=1` is set.
7. Print a one-line summary to the user: `Agile Loop: repo=<repo> base=<base> max=<n> dry_run=<bool> run=<RUN_ID> dashboard=<url|skipped>`.

## Child-session invocation

For every agent-backed step, spawn one fresh child via `Bash`, with its working directory set to the current task's worktree (`$WORKTREE`, created during claim):

```bash
( cd "$WORKTREE" && claude -p "$(cat "$RUN_DIR/<stage>.prompt.md")" \
  --model <stage-model> \
  --output-format text \
  --dangerously-skip-permissions \
  > "$RUN_DIR/<stage>.output.md" \
  2> "$RUN_DIR/<stage>.events.log" )
```

- The child runs **inside the task's Claude worktree** (`cd "$WORKTREE"`), so all of its file edits and commits land on the task branch and never touch the main checkout. The prompt `cat`, stdout `>`, and stderr `2>` all use **absolute** `$RUN_DIR` paths, so run artifacts stay in the main repo even though cwd is the worktree.
- `--dangerously-skip-permissions` is required so the child can write files and run shell commands without prompting. Only pass it when the parent loop is gated by `--unsafe-bypass-approvals` / `AGILE_LOOP_UNSAFE_BYPASS=1`. In `--dry-run` mode, do not spawn the child at all — log "DRY RUN: would run stage=<stage>".
- Both `--model <stage-model>` and `--effort <stage-effort>` are mandatory on **every** stage — never let a stage silently inherit an ambient default model or effort. Resolve both from the stage's row in **Model and effort routing** below and pass them explicitly on each `claude -p` call.
- Build each prompt file first using the inline templates in **Prompt templates**.

## Model and effort routing

Each stage pins its own model and effort — there is no single default. Most stages run on the latest Opus; the ship and auto-merge stages run on the latest Sonnet. Effort is set per stage (`max`, `xhigh`, or `medium`) and, for deep review, varies by pass.

- **Plan → latest Opus, `--effort max`.** Full power deciding the approach up front.
- **Implementation → latest Opus, `--effort medium`.**
- **Deep review pass 1 → latest Opus, `--effort xhigh`.** Deepest review pass.
- **Deep review pass 2 → latest Opus, `--effort medium`.** Lighter confirmation pass.
- **All remediation + QA (remediate-deep-review pass 1 & 2, qa, remediate-qa) → latest Opus, `--effort medium`.**
- **Ship and auto-merge → latest Sonnet, `--effort max`.**

"Latest Opus" is the `opus` model alias (currently `claude-opus-4-8`); "latest Sonnet" is the `sonnet` alias (currently `claude-sonnet-4-6`). When a newer release ships, the stages follow the alias automatically.

| Tier          | How to pass it                                   |
| ------------- | ------------------------------------------------ |
| Latest Opus   | `--model opus` (currently `claude-opus-4-8`)     |
| Latest Sonnet | `--model sonnet` (currently `claude-sonnet-4-6`) |

Per-stage routing (`<stage-model>` / `<stage-effort>` for each `claude -p` invocation):

| Stage (`stage` value)          | `--model`         | `--effort` |
| ------------------------------ | ----------------- | ---------- |
| `plan`                         | `opus` (latest)   | `max`      |
| `implement`                    | `opus` (latest)   | `medium`   |
| `deep-review` (pass 1)         | `opus` (latest)   | `xhigh`    |
| `deep-review` (pass 2)         | `opus` (latest)   | `medium`   |
| `remediate-deep-review` (both) | `opus` (latest)   | `medium`   |
| `qa`                           | `opus` (latest)   | `medium`   |
| `remediate-qa`                 | `opus` (latest)   | `medium`   |
| `ship`                         | `sonnet` (latest) | `max`      |

`deep-review` uses `xhigh` on pass 1 (`{pass}=1`) and `medium` on pass 2 (`{pass}=2`); branch on the pass when assembling the invocation. The auto-merge step (step 10) is a mechanical `gh pr merge` run by the orchestrator, not a `claude -p` child, so it has no model/effort knob — the "ship and auto-merge → Sonnet" routing applies only to the `ship` child that precedes it.

The Codex adapter (`scripts/agile-loop.sh`) passes `--default-model` / `--implementation-model` / `--review-model` for equivalent routing.

## Per-iteration loop

For up to `--max-iterations` iterations, or until the queue is empty:

### 1. Pick the next task

- `Glob docs/agile-loop/tasks/*.md` sorted lexically.
- Read each file's YAML frontmatter. Pick the first with `status: todo`. If none, write `status: idle` and stop with a one-line summary.
- Edit the chosen task file's frontmatter to `status: doing`. Write status.json with `stage: claim`, `stage_status: running`, the task path in `task_file`, and the current iteration number. In `--dry-run`, do not mutate the task file; just log "would claim <task>".
- **Create the task's Claude worktree** (still under `stage: claim`; this git op runs in the main repo). Derive a `slug` from the task-file stem: lowercase it, replace every `[^a-z0-9-]` with `-`, collapse repeats, and trim leading/trailing `-`. Set `WORKTREE=$WORKTREES/agile-loop-<slug>` and `BRANCH=claude/agile-loop-<slug>`, then `mkdir -p "$WORKTREES"` and `git worktree add -b "$BRANCH" "$WORKTREE" "<base>"`. Every agent stage for this task runs with cwd `$WORKTREE` (see **Child-session invocation**).
  - **Resume / collision:** if `$WORKTREE` already exists and is checked out on `$BRANCH` (e.g. a task set back to `todo` after a prior block), reuse it; recreate only if it is missing or corrupt (`git worktree remove "$WORKTREE" --force`, then re-add). If a _different_ worktree or branch already squats the name, append a short `$RUN_ID` suffix to both `WORKTREE` and `BRANCH` so the pair is unique.
  - All task-frontmatter edits (`doing` / `done` / `blocked`) happen on the **main-repo** task file, never the worktree copy — so task-status churn never enters the PR diff and the dashboard queue glob stays accurate. No child stage edits the task frontmatter; only the orchestrator does.
  - In `--dry-run`, create nothing; log `DRY RUN: would create worktree $WORKTREE on branch $BRANCH`.

### 2. Run the 10-step loop contract

The steps (identical contract to `scripts/agile-loop.sh`):

1. **Plan** — `Planning Session` template. Convert the task into a Superpowers plan.
2. **Implement** — `Implementation Session` template. Drive the full Superpowers implementation discipline: `subagent-driven-development` (per-task implement + two-stage review) with `test-driven-development` per task, `systematic-debugging` when stuck, and a `verification-before-completion` gate. Stop before `finishing-a-development-branch` — shipping is step 9.
3. **Deep review pass 1** — `Deep Review Session` template, `{pass}=1`. Parse the final JSON line `{"critical","major","minor","blocked","summary"}`.
4. **Remediate deep review** — `Deep Review Remediation Session` template. Only spawn if pass 1 has `critical>0 || major>0`. Pass the pass-1 output file path as `{review_output}`.
5. **QA** — `QA Session` template. Parse final JSON line `{"issues","blocked","report"}`.
6. **Remediate QA** — `QA Remediation Session` template. Only if `issues>0`. Pass the QA output path as `{qa_output}`.
7. **Deep review pass 2** — `Deep Review Session` template, `{pass}=2`. Parse the same JSON shape.
8. **Remediate deep review pass 2** — only if pass 2 has `critical>0 || major>0`.
9. **Ship** — `Ship Session` template. Parse the final JSON line `{"blocked","pr_url","summary"}`. Capture `pr_url`.
10. **Auto-merge** — the loop merges the PR itself; see **Auto-merge handling** below. Do not wait for a human.

Spawn each step's child with the `--model` and `--effort` from **Model and effort routing**: plan on Opus `max`; implement on Opus `medium`; deep-review on Opus `xhigh` (pass 1) / `medium` (pass 2); remediation and QA on Opus `medium`; ship on Sonnet `max`. Update status.json before and after each step. Use `stage` values `plan`, `implement`, `deep-review`, `remediate-deep-review`, `qa`, `remediate-qa`, `ship`. Every one of these stages spawns with cwd set to the task's worktree (`$WORKTREE`); prompt files, stage outputs, and `status.json` stay in the main repo via `$RUN_DIR`.

### Retry policy per agent-backed step

- 3 exponential-backoff retries (5s, 10s, 20s) on: child non-zero exit, empty stdout, JSON-contract stages missing the final JSON line, transient parse errors. Update status with `stage_status: retrying` between attempts.
- Do NOT retry on: explicit `BLOCKED` or `NEEDS_CONTEXT` in the child output, or `blocked: true` in a JSON-contract stage's final line. Stop immediately: write `status: blocked` with the child's reason, flip the task file frontmatter back to `status: blocked` (writing the reason into the task file body as a `## Blocked` section), and exit the loop. **Leave the task worktree in place** for inspection — do not remove it on a block. Include `Worktree: $WORKTREE (branch $BRANCH)` in the `status: blocked` `message` and in the task's `## Blocked` section so a human can `cd` in; the next run reuses it (see **Pick the next task**). This leave-in-place rule applies to every terminal block below (failed merge, failed fast-forward, exhausted retries), not just child-reported blocks.
- For non-JSON stages, the final `STATUS:` line drives branching. `DONE` and `DONE_WITH_CONCERNS` continue; `BLOCKED` and `NEEDS_CONTEXT` stop.
- **Usage-limit auto-resume**: if a child exits non-zero and either stdout or the events log (`.agile-loop/runs/$RUN_ID/<stage>.events.log`) contains any of the strings `usage limit`, `rate limit`, or `overloaded` (case-insensitive), treat it as a time-gated pause — not a permanent failure and NOT counted against the 3-retry budget:
  1. Write `status: waiting`, `stage_status: waiting`, `message: "Usage limit hit; will retry at HH:MM UTC"`.
  2. Compute seconds until top of next UTC hour (add 30s buffer): `python3 -c "import time; t=time.time(); print(int(3600 - t % 3600 + 30))"`. Sleep that long via `Bash`.
  3. After waking, write `status: running`, `stage_status: running`, then re-spawn the same stage from scratch (the child is stateless — full re-spawn is safe).
  4. Repeat indefinitely on consecutive usage-limit hits. Only promote to a normal retry (against the 3-retry budget) when the failure is NOT a usage-limit string.

### 3. Auto-merge handling

The ship stage (step 9) runs in the task worktree on `$BRANCH`, so `/ship` pushes that branch and opens the PR against `<base>` — nothing extra is needed. After step 9 writes `pr_url`, the loop merges the PR itself (back in the main repo) — there is no human merge gate and no polling:

1. Write `status: running`, `stage: merge`. Squash-merge the PR with admin override and branch cleanup: `gh pr merge <pr_url> --squash --admin --delete-branch`. `--admin` forces past branch protection / required checks so the loop never blocks waiting on a reviewer or CI gate. Retry the merge on transient failure with the same budget as every other step (3 attempts, exponential backoff 5s, 10s, 20s — see **Retry policy per agent-backed step**). If `gh pr merge` exits non-zero, do not block immediately: re-check the PR's real state with `gh pr view <pr_url> --json state`. If `state` is `MERGED`, treat it as success and continue — this is common when the repo auto-deletes head branches, so `--delete-branch` errors on an already-gone branch; only log a warning that the branch cleanup failed. Set `status: blocked` (reason e.g. "failed to squash-merge PR <pr_url>") and stop ONLY if the PR is genuinely not `MERGED` after retries are exhausted. In `--dry-run`, do not merge; log "DRY RUN: would run gh pr merge <pr_url> --squash --admin --delete-branch".
2. Tear down the task worktree, then sync the base branch. Write `stage: sync-base`, then run these in the main repo (`$REPO`):
   - **Remove the worktree first**: `git worktree remove "$WORKTREE" --force`. This must precede the local-branch delete because `$BRANCH` is checked out in the worktree and cannot be deleted while it exists — which is also why `gh pr merge --delete-branch` could only ever remove the _remote_ head, not the local branch.
   - Delete the now-free local branch: `git branch -D "$BRANCH"`. Then `git worktree prune`. These three teardown commands are **non-blocking** — on failure, log a warning and continue (same stance as the branch-cleanup-failure case above).
   - `git fetch origin <base>`.
   - `git checkout <base>` (or `git switch <base>`).
   - Fast-forward only: `git merge --ff-only refs/remotes/origin/<base>`. If the fast-forward fails, set `status: blocked` with reason "base branch <base> diverged from origin; cannot fast-forward" and stop. **Do not rebase or force-update.**
   - In `--dry-run`, do not remove anything; log `DRY RUN: would remove worktree $WORKTREE`.
3. Edit the task file's frontmatter to `status: done`. Write status.json with `stage: complete`, `stage_status: completed`.
4. If iterations remain and the queue has more `todo` tasks, continue to the next iteration. Otherwise stop with `status: idle`.

## Prompt templates

Write the assembled template to `.agile-loop/runs/$RUN_ID/<stage>.prompt.md` before invoking `claude -p`. Each template ends with the same `Session isolation:` block and the exit contract (a final `STATUS:` line or a final JSON line, depending on the stage). Substitute `{repo}` with the absolute repo path, `{workdir}` with the absolute path to the task's Claude worktree (the working directory every stage runs in), `{base}` with the base branch, `{task_file}` with the task file path (the main-repo copy — read-only context), and the remediation-specific keys (`{pass}`, `{review_output}`, `{qa_output}`, `{max_parallel_remediation}` — default `6`).

The shared `Session isolation:` block (appended to every template):

```
Session isolation:
This is a fresh agent session for exactly this Agile Loop stage. Reconstruct all context from the repository, task file, and referenced stage output files. Do not rely on previous child-session chat history. Do not resume any prior session.
```

### Planning Session

```
You are running agile-loop stage: plan.

Repository: {repo}
Working directory: {workdir} (this is a git worktree — do all work here; do not cd elsewhere)
Base branch: {base}
Task file: {task_file}

<<session-isolation>>

Read AGENTS.md and the task file first. Use the Superpowers writing-plans skill to convert the task into a decision-complete implementation plan. Save the plan in the repo's normal Superpowers plan location unless the task file specifies a stronger location.

Stop with BLOCKED if the task lacks enough objective or acceptance context to plan safely.
End with:
STATUS: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
```

### Implementation Session

```
You are running agile-loop stage: implement.

Repository: {repo}
Working directory: {workdir} (this is a git worktree — do all work here; do not cd elsewhere)
Base branch: {base}
Task file: {task_file}

<<session-isolation>>

Drive this stage through the installed Superpowers plugin's full implementation discipline. Invoke each as a Skill (engage the installed plugin — do not merely imitate the workflow):

1. superpowers:subagent-driven-development — execute the current Superpowers plan task-by-task: a fresh implementer subagent per task, then the two-stage spec-compliance then code-quality review it prescribes, and a final whole-implementation review at the end.
2. superpowers:test-driven-development — implementer subagents write a failing test and watch it fail before any production code (RED → GREEN → REFACTOR). No production code without a failing test first.
3. superpowers:systematic-debugging — when a test fails or behavior is unexpected, root-cause it with this skill instead of guessing.
4. superpowers:verification-before-completion — before reporting DONE, run the plan's verification commands fresh and confirm the output. Evidence before claims.

Keep all changes scoped to the plan. Per-task commits on the working branch are expected.

Do not cross the ship boundary: do NOT run superpowers:finishing-a-development-branch, do NOT open a PR, and do NOT merge or push to the base branch. Review, QA, and ship are later agile-loop stages — leave the work committed on the branch and stop.

Stop with BLOCKED if tests fail and cannot be fixed inside the task scope, if mandatory user judgment is needed, or if the plan is missing.
End with:
STATUS: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
```

### Deep Review Session

```
You are running agile-loop stage: deep review pass {pass}.

Repository: {repo}
Working directory: {workdir} (this is a git worktree — do all work here; do not cd elsewhere)
Base branch: {base}
Task file: {task_file}

<<session-isolation>>

Use the built-in /code-review skill at high effort to review the current branch's diff against {base}. Pass AGENTS.md as additional review context when present. This stage is report-only: do not apply fixes.

Classify each finding by severity:
- critical: correctness/security defects unsafe to merge or that break the feature.
- major: likely bugs, missing error handling, or significant design problems.
- minor: style, naming, small cleanups, or non-blocking suggestions.

Summarize issues by severity. The final line of your response must be exactly one JSON object:
{"critical":0,"major":0,"minor":0,"blocked":false,"summary":"short summary"}
```

### Deep Review Remediation Session

```
You are running agile-loop stage: remediate deep review.

Repository: {repo}
Working directory: {workdir} (this is a git worktree — do all work here; do not cd elsewhere)
Base branch: {base}
Task file: {task_file}
Review output file: {review_output}
Maximum remediation sub-agents: {max_parallel_remediation}

<<session-isolation>>

Read the deep-review output. Only if it contains Critical or Major issues, spawn scoped sub-agents to fix those issues. Keep each sub-agent's write scope disjoint and tied to one finding or file group. Do not fix Minor issues unless they are necessary for a Critical or Major fix.

Run targeted validation for the changed files. End with:
STATUS: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
```

### QA Session

```
You are running agile-loop stage: qa-only full.

Repository: {repo}
Working directory: {workdir} (this is a git worktree — do all work here; do not cd elsewhere)
Base branch: {base}
Task file: {task_file}

<<session-isolation>>

Use /qa-only with mode: full. This stage is report-only: do not fix. Prefer the running local app and the task/plan verification steps. Include paths to the QA report.

The final line of your response must be exactly one JSON object:
{"issues":0,"blocked":false,"report":"path or short summary"}
```

### QA Remediation Session

```
You are running agile-loop stage: remediate qa.

Repository: {repo}
Working directory: {workdir} (this is a git worktree — do all work here; do not cd elsewhere)
Base branch: {base}
Task file: {task_file}
QA output file: {qa_output}
Maximum remediation sub-agents: {max_parallel_remediation}

<<session-isolation>>

Read the QA report. Only if it contains issues, spawn scoped sub-agents using /investigate to root-cause and fix them. Keep each fix scoped to a reproducible QA issue.

Rerun the relevant validation for fixed issues. End with:
STATUS: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
```

### Ship Session

```
You are running agile-loop stage: ship.

Repository: {repo}
Working directory: {workdir} (this is a git worktree — do all work here; do not cd elsewhere)
Base branch: {base}
Task file: {task_file}

<<session-isolation>>

Use /ship. Run the full ship workflow. Stop with BLOCKED if tests fail, review requires human judgment, auth is missing, or the PR cannot be created.

The final line of your response must be exactly one JSON object:
{"blocked":false,"pr_url":"https://github.com/owner/repo/pull/123","summary":"short ship summary"}
```

Replace `<<session-isolation>>` in every template with the shared block above.

## Status writer

Both adapters write `.agile-loop/status.json` with this schema. **These field names are canonical** — the Codex writer (`scripts/agile-loop.sh::write_status`) and the dashboard (`scripts/agile-dashboard.py`) read these exact keys, so the Claude adapter must use them verbatim. Earlier wording that referred to `loop_status` or `task` was wrong; use `status` and `task_file`.

```json
{
  "run_id": "<RUN_ID>",
  "repo": "<absolute repo path>",
  "base": "<base branch>",
  "status": "starting|running|waiting|blocked|idle|dry_run",
  "stage": "claim|plan|implement|deep-review|remediate-deep-review|qa|remediate-qa|ship|merge|sync-base|complete|dashboard",
  "stage_status": "running|completed|failed|retrying|blocked|waiting|skipped",
  "message": "human-readable summary",
  "task_file": "<absolute path to task file, or empty>",
  "task_title": "<title from task frontmatter, or task file stem>",
  "iteration": 1,
  "pr_url": "https://github.com/owner/repo/pull/123",
  "dry_run": false,
  "run_dir": "<absolute path to .agile-loop/runs/$RUN_ID>",
  "log_file": "<absolute path to loop log>",
  "updated_at": "2026-06-08T17:00:00Z"
}
```

Write status atomically — temp file under `.agile-loop/` then `mv`. The canonical implementation is `write_status()` in `scripts/agile-loop.sh`; mirror its field names and behavior exactly. A small `python3 -c` heredoc invoked via `Bash` is sufficient — pass each value as an argv arg and let Python build the dict.

## Dashboard auto-start

The dashboard (`scripts/agile-dashboard.py`) is part of the agile-loop user contract: every time the skill is used, `http://<dashboard-host>:<dashboard-port>` (default `http://127.0.0.1:8765`) MUST be serving the current `--repo`. Do not assume the user has it running — start or reclaim it during pre-flight.

Algorithm — run via `Bash` once, after the initial `status: starting` write and before the per-iteration loop:

1. If `--no-dashboard` (or `AGILE_LOOP_NO_DASHBOARD=1`) is set: log "dashboard auto-start disabled" and skip. The user is on the hook for running their own.
2. Probe the URL: `curl -fsS --max-time 2 http://<host>:<port>/api/status`. Parse the JSON `repo` field.
   - If it equals the current `--repo` (resolved absolute path): log "dashboard already serving <repo>" and skip — idempotent reuse, do not respawn.
   - If it returns a _different_ repo: a stale dashboard from a prior run is squatting on the port. Reclaim it: find the listener with `lsof -ti tcp:<port> -sTCP:LISTEN`, `kill` then `kill -9` after a 1s grace, then proceed to step 3. Log the reclamation so the user knows.
   - If the probe fails (connection refused / timeout): proceed to step 3.
3. Spawn the dashboard detached so it outlives the loop run:

   ```bash
   nohup python3 scripts/agile-dashboard.py \
     --repo "$REPO" \
     --status-file .agile-loop/status.json \
     --queue-glob 'docs/agile-loop/tasks/*.md' \
     --host "$DASHBOARD_HOST" \
     --port "$DASHBOARD_PORT" \
     > .agile-loop/runs/$RUN_ID/dashboard.log 2>&1 &
   echo "$!" > .agile-loop/dashboard.pid
   disown 2>/dev/null || true
   ```

   Use `nohup` (or a detached subshell + `disown`) so the dashboard process is NOT a child of the current Bash invocation — when the loop completes, the dashboard keeps running so the user can still see the final state.

4. Wait ~1s, then re-probe `/api/status` once to confirm the spawn succeeded. If still unreachable, write a non-blocking warning to the log (`dashboard did not respond at <url> within 1s; check <log>`) and continue — a missing dashboard is annoying but not blocking.

Never edit `scripts/agile-dashboard.py` to "show" tasks differently — it reads disk on every poll. Keeping the dashboard current means keeping `.agile-loop/status.json` and `docs/agile-loop/tasks/*.md` current.

The Codex adapter (`scripts/agile-loop.sh::ensure_dashboard`) implements the same algorithm with the same flag names and same default port; both adapters must stay in sync.

## Guardrails (non-negotiable)

- **Notification policy: only notify on failure.** Do NOT send user-visible push notifications (e.g. via the `PushNotification` tool) on routine stage transitions, retries, task claims, successful merges, or queue-empty completion. Only notify when the loop enters a terminal `status: blocked` state (failed merge, failed fast-forward, child reported `BLOCKED` / `NEEDS_CONTEXT`, exhausted retries on a non-usage-limit error, or any other condition that stops the loop without progress). Status writes to `.agile-loop/status.json` and the dashboard are NOT notifications — keep updating them on every stage as before. Usage-limit pauses (`status: waiting`) are not failures — do not notify on those either.
- One PR per queued task.
- **Per-task worktree isolation.** Each task runs in its own Claude worktree (`.claude/worktrees/agile-loop-<slug>` on branch `claude/agile-loop-<slug>`); every agent stage runs inside it and never in the main checkout. Orchestration (status.json, task-frontmatter edits, `gh` merge, base sync) stays in the main repo. Only the orchestrator edits task frontmatter, and only the main-repo copy — no child stage touches it.
- **Worktree teardown only after a successful merge.** Remove the worktree before deleting its local branch (the branch is checked out in the worktree). Teardown (`git worktree remove` / `git branch -D` / `git worktree prune`) is non-blocking — warn and continue on failure. On any terminal block, leave the worktree in place for inspection and record its path in the status `message` and the task's `## Blocked` section.
- Stop instead of guessing on: `BLOCKED`, `NEEDS_CONTEXT`, failed tests, mandatory user judgment, missing authentication, or a failed auto-merge.
- Delegate review and QA fixes only after findings exist (no preemptive cleanup).
- Keep remediation scoped to the finding source. Do not broaden into cleanup.
- Prefer repo guidance from `AGENTS.md` when present in the target repo.
- Auto-merge each task's PR (`gh pr merge --squash --admin --delete-branch`) — do not wait for a human and do not poll. Retry the merge on transient failure, then re-check the PR's real state: a PR that is genuinely not `MERGED` after retries are exhausted blocks the loop, but a successful merge whose branch cleanup failed (e.g. the head branch was already auto-deleted) does NOT block — just warn and continue. The loop proceeds once the PR is merged and the base branch is synced.
- Do not mark a task `done` until the PR is merged and the configured base branch has successfully fast-forwarded to `origin/<base>`. Do not use a post-merge rebase to replay local base-branch commits.
- `--dry-run` prints what would happen and writes `status: dry_run` / `stage_status: skipped` for each stage. Do not spawn child sessions, do not mutate task files, do not run `git fetch` or `gh` write operations, and do not run `git worktree add/remove` or `git branch -D` — log the intended worktree actions only.
- Live runs must be opted into with `--unsafe-bypass-approvals` (or `AGILE_LOOP_UNSAFE_BYPASS=1`); otherwise refuse to spawn child sessions.

## Queue format

Each `docs/agile-loop/tasks/*.md` file uses this shape:

```markdown
---
status: todo|doing|done|blocked
title: Short task title
phase: optional-gsd-phase-id
---

## Objective

What to build.

## Inputs

Links to GSD phase docs, plans, screenshots, issues, or acceptance notes.

## Done

Concrete acceptance criteria.
```

## References

- Prompt contracts (host-neutral documentation): `references/prompts.md`. Both adapters inline equivalent templates at runtime; that file is the spec, not a runtime dependency.
- Codex adapter (reference implementation for retry / JSON parsing / status writes): `scripts/agile-loop.sh`.
- Dashboard: `scripts/agile-dashboard.py` (defaults to `http://127.0.0.1:8765`; reads `.agile-loop/status.json`). Auto-started by both adapters during pre-flight — see **Dashboard auto-start**.
- Codex-side metadata: `agents/openai.yaml`. Claude-side metadata: `agents/claude.yaml`.

---
> Source: [nirmitgoyal/agile-loop](https://github.com/nirmitgoyal/agile-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
