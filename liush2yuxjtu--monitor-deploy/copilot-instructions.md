## monitor-deploy

> Tail runtime logs from any deployment platform (Vercel / file / stdin / custom command), group by a configurable request-id regex, classify per a project-supplied known-bug ruleset, dedupe via signature hash, surface NEW bugs as macOS notifications + auto-filed `.agent/issues/auto-NNNN-*.md`, write lighter case + anomaly records for known-bug hits + out-of-scope findings into `.agent/issues/case-NNNN-*.md` and `.agent/issues/anomaly-NNNN-*.md`, and append to `.agent/monitor-rollup.md`. Auto-INDEX.md tracks every file written. Pairs with `/loop 5m /monitor-deploy` for continuous tail. Cheap to no-op: most ticks fire 0 new bugs. **This is the project-agnostic version** — see `monitor-deploy.config.example.yaml` for wiring.


# /monitor-deploy — generic runtime-log watcher

> **Project-agnostic.** Originally written for a Vercel codex-pdf deploy; refactored so any project can drop it in, point it at their runtime logs, supply a request-id regex + a seed of known signatures, and get the same auto-classify / dedupe / surface loop. See `monitor-deploy.config.example.yaml` for the full config schema.

The skill watches runtime logs autonomously, classifies failures per a known-bug ruleset, and surfaces NEW signatures the moment they appear — so deploy bugs go from "user reports broken canary" → "issue auto-filed for `/afk-agents`" without a human in the loop.

## When to invoke

- One-shot: `/monitor-deploy` — does one polling tick and exits
- Continuous: `/loop 5m /monitor-deploy` — fires every 5 min. Cheap to no-op; most ticks find nothing new.
- The skill is **idempotent**: re-running with the same state log + the same logs produces zero new alerts.

## Inputs (optional positional args)

- **First arg**: log source override. If unset, uses `log_source` from `monitor-deploy.config.yaml` (resolved relative to the cwd).
- **Second arg**: lookback window (e.g. `15m`, `1h`). Default = `since last_seen_ts in .agent/monitor-state.json` OR `default_lookback` from config (default `15m`) on first invocation.

## Configuration

The skill reads `monitor-deploy.config.yaml` from the cwd. **All project-specific knobs live in the config**, not in this file. See `monitor-deploy.config.example.yaml` for the full schema with sensible defaults.

Minimal example for a Vercel project:

```yaml
log_source:
  type: vercel
  project_id: null              # falls back to .vercel/project.json:projectId
  team_id: null                 # falls back to .vercel/project.json:orgId
  environment: production
  limit: 100
reqid_pattern: '\[(job|api|worker) ([a-z0-9]{6})\]'
slow_threshold_ms: 30000
default_lookback: 15m
notification:
  enabled: true
  title: "<project-slug>: new bug signature"
  sound: Funk
issues_dir: .agent/issues
state_file: .agent/monitor-state.json
rollup_file: .agent/monitor-rollup.md
rollup_max_lines: 500
```

Minimal example for a self-hosted Docker / k8s project using `kubectl logs`:

```yaml
log_source:
  type: command
  cmd: "kubectl logs -n prod -l app=api --since=<lookback> --tail=200"
  # The skill will append `--since=<lookback>` if your cmd includes the literal <lookback>
reqid_pattern: '\[([a-z\-]+) ([a-z0-9]{6,12})\]'
```

Minimal example for a project whose logs land on disk:

```yaml
log_source:
  type: file_tail
  paths: ["./logs/api.log", "./logs/worker.log"]
  # Skill tracks per-file byte offsets in state; new lines per tick are processed.
```

If no config exists, the skill fails fast with a one-liner pointing to `monitor-deploy.config.example.yaml` — it does NOT silently fall back to Vercel defaults.

## What the skill does, top to bottom

1. **Read state** from `<state_file>` (default `.agent/monitor-state.json`):
   ```json
   { "last_seen_ts": "2026-06-13T01:35:00Z", "tick_count": 12, "total_alarms": 0, "new_signatures_this_session": 0 }
   ```
   If the file doesn't exist, treat `last_seen_ts` as `default_lookback` ago and create the file at the end.

2. **Read known signatures** from `.agent/bug-signatures.json`:
   ```json
   {
     "sig:<your-slug>": {
       "first_seen": "2026-06-12T11:00:00Z",
       "hit_count": 9,
       "pattern": "<known-bad substring or regex>",
       "memo": "<human-readable explanation + suggested fix>"
     }
   }
   ```
   **This file is project-owned.** The skill seeds it with whatever the project supplies; it does NOT bundle defaults. The 4 seed examples that ship in this skill's `examples/bug-signatures.example.json` are for a Vercel PDF render — copy them as a starting point or replace with your own.

3. **Pull runtime logs** via the configured `log_source`:
   - `type: vercel` → call `mcp__plugin_vercel_vercel__get_runtime_logs` (needs the Vercel MCP loaded; falls back to "platform unavailable" anomaly if MCP absent)
   - `type: file_tail` → `tail -n 200 <paths> | sort -k1,1` (offset persisted in state for incremental reads)
   - `type: command` → `bash -lc "<cmd>"`; if the cmd string contains the literal `<lookback>`, the skill substitutes the resolved window
   - `type: stdin` → read all of stdin (for piping from a curl in a shell pipeline)
   - Don't pass a `level` filter — the classifier needs info-level lines too, so it can group by reqId.

4. **Group lines by reqId.** For each line, apply the configured `reqid_pattern` (default `\[(job|api|worker) ([a-z0-9]{6})\]`). Bucket every line whose reqId matches into one job. Lines without a reqId (raw tracebacks, request-summary lines) go into a `_unbound` bucket — these are surfaced as anomalies, not as classified jobs.

5. **Classify each completed job**. Every classified job writes a record to `<issues_dir>` so the cases are auditable and `/afk-agents` (or `/office-agents`) can later process them:
   - **success (fast)**: total_ms < `slow_threshold_ms` → just a one-line entry in `<rollup_file>`, no file. Don't drown the issues dir in healthy-run noise.
   - **success (slow / anomalous)**: total_ms ≥ `slow_threshold_ms` OR any phase looks broken (e.g. one phase alone > 50% of threshold) → write `<issues_dir>/case-NNNN-<reqId>-success.md` (light template below) AND rollup entry. The case file is the "why was this so slow?" audit trail.
   - **known-bug**: any line matches a `pattern` from bug-signatures.json → increment `hit_count` in bug-signatures.json (silent — no osascript), write `<issues_dir>/case-NNNN-<reqId>-known-<sig>.md` (light template), append to rollup as known hit, NO alarm. The case file makes every known-bug occurrence auditable even if the rollup gets rotated.
   - **out-of-scope finding**: a non-runtime-log signal observed in the same tick (deploy state in ERROR, build logs truncated, snapshot env drifted, log source unreachable, etc.) → write `<issues_dir>/anomaly-NNNN-<short-slug>.md` (anomaly template below) AND note it under a `### ⚠ Out-of-scope-but-critical findings` heading in the rollup. Operator surface only; not auto-dispatched by `/afk-agents` because `/afk-agents` filters on `triage: ready-for-agent`.
   - **NEW signature**: line has `level=error` or `status >= 500` AND doesn't match any known pattern → THIS IS THE INTERESTING CASE:
     1. Fingerprint = `(error-class, first 80 chars of msg, route)` → SHA-1 truncated to 12 chars
     2. If `sig:auto-<fingerprint>` already exists in bug-signatures.json → known, no alarm
     3. Otherwise — file a new auto-issue. The NNNN is monotonic across all auto-*.md files in the issues dir; compute the next free id BEFORE writing the file so a second concurrent tick can't collide:
        ```bash
        next_n=$(ls <issues_dir>/auto-*.md 2>/dev/null \
          | sed -E 's|.*/auto-([0-9]+)-.*|\1|' \
          | sort -n | tail -1 | awk '{print $1+1}')
        [ -z "$next_n" ] && next_n=1
        ```
     4. With `next_n` decided, do all of these in order (failures in step 5.4-5.6 should still leave step 5.3's signature in `bug-signatures.json` — don't roll it back, the human can re-file):
        - Add new signature to `bug-signatures.json` (`first_seen` = now, `hit_count` = 1, `pattern` = first 80 chars, `memo` = "AUTO-DETECTED — needs triage")
        - Auto-file an issue at `<issues_dir>/auto-NNNN-<short-slug>.md` with the full error context (template below)
        - Fire macOS notification: `osascript -e 'display notification "<msg snippet>" with title "<project notification title>" sound name "<sound>"'`
        - Increment `new_signatures_this_session` in state
     5. **Append to `<issues_dir>/auto-INDEX.md`** so `/afk-agents` + `/office-agents` and any human reading the project see the new auto-issue without `ls`-ing the dir. The auto-INDEX is a flat append-only table — one row per file written this tick (auto / case / anomaly), grouped under a `## Tick @ <ts>` heading, with a `kind` column so the reader can filter:
        ```
        ## Tick @ 2026-06-13T01:35Z
        | kind | id | first_seen | sig | title (truncated to 60) | file |
        |---|---|---|---|---|---|
        | auto | auto-0001 | 2026-06-13T01:20:00Z | sig:auto-a1b2c3d4e5f6 | TypeError: Cannot read property 'x' of undefined... | auto-0001-type-err-pdf-file.md |
        | case | case-0001 | 2026-06-13T01:35:00Z | sig:known-sig-id | pdf abc123 hit known sig (took 312s) | case-0001-abc123-known-known-sig-id.md |
        | anomaly | anomaly-0001 | 2026-06-13T01:36:00Z | — | log source unreachable | anomaly-0001-log-source-unreachable.md |
        ```
        If `auto-INDEX.md` doesn't exist yet, create it with the header row (a single empty `## Tick @ <bootstrap ts>` section) before appending. `/afk-agents` reads `*.md` from the issues dir, but the file's `triage:` field gates dispatch (auto-issues use `ready-for-agent`, cases use `in-review` (already-known), anomalies use `blocked` (operator-only) — so `/afk-agents` naturally picks up only the auto rows).

6. **Append to `<rollup_file>`** — one-line entries per job seen this tick (success / known / new), grouped under a `## Tick @ <ts>` heading. Keep the file under `rollup_max_lines`: if it grows past the cap, prepend `<!-- truncated, see <rollup_file>-archive.md -->` and rotate the oldest 200 lines into a sibling archive.

7. **Update state**:
   - `last_seen_ts` ← max(`ts` over all lines pulled, current time if no lines)
   - `tick_count` ← `tick_count + 1`
   - `total_alarms` ← `total_alarms + new_signatures_this_session`

8. **Print summary** to stdout:
   ```
   monitor-deploy — tick N | seen=42 lines | jobs=7 (✓5, known-bug=2, new=0) | total alarms to date=0
   ```
   If new bugs were fired, list them AND surface the exact next-action so the parent doesn't have to `ls` the issues dir to figure out what to do:
   ```
   monitor-deploy — tick N | ⚠ 1 NEW BUG: sig:auto-a1b2c3d4 "TypeError: Cannot read property..." → .agent/issues/auto-0001-type-err-pdf-file.md → run /afk-agents to dispatch
   ```
   With multiple new bugs, list them as a stacked block and end with a single `→ run /afk-agents to dispatch` hint (don't repeat the hint per line).

9. **Exit the turn.** The user (or `/loop`) re-invokes. Each re-invoke is independent.

## Case record template (success-slow + known-bug)

Used by `case-NNNN-<reqId>-<kind>-<short-sig>.md`. Light: enough context to audit the run later, not a full fix-this-please auto-issue.

```yaml
---
id: "case-<NNNN>"
title: "CASE: <kind> · <reqId> · <one-line summary>"
kind: case
priority: P3
estimate_hours: 0
status: closed
owner: monitor-deploy
triage: in-review
labels: [monitor-deploy, <kind>]
acceptance: []
---

## Run summary

- **reqId**: <reqId>
- **Route**: <route>
- **Total**: <total_ms>ms
- **Phases**: <phase-name=ms · phase-name=ms · …>
- **First observed**: <ISO ts>
- **Tick**: <tick_count>

## Classification

- **kind**: success-slow | known-bug
- **Signature (if known-bug)**: <sig:id> — <memo from bug-signatures.json>
- **hit_count after this run**: <N>

## Lines (chronological, this tick only)

```
<every line in the bucket, prefixed with its time>
```

## Why this file exists

A routine observed case — not a NEW bug (those go to `auto-NNNN-*.md`). Written so the audit trail is searchable by reqId even after `<rollup_file>` rotates. If a known-bug signature escalates (e.g. hit_count goes 5→50 in 24h), the human can pull all `case-*.md` rows for that signature and decide to file a real fix slice via `/to-issues`.
```

NNNN derives from `ls <issues_dir>/case-*.md | sed -E 's|.*/case-([0-9]+)-.*|\1|' | sort -n | tail -1 | awk '{print $1+1}'`, default 1.

## Anomaly record template (out-of-scope findings)

Used by `anomaly-NNNN-<short-slug>.md`. Same YAML shape as the case record but with `kind: anomaly`, `triage: blocked` (operator-only), and a body that explains the signal channel and the operator action.

```yaml
---
id: "anomaly-<NNNN>"
title: "ANOMALY: <one-line summary of the out-of-scope finding>"
kind: anomaly
priority: P2
estimate_hours: 0.25
status: pending
owner: unassigned
triage: blocked
labels: [monitor-deploy, anomaly, <channel-tag>]
acceptance:
  - operator triages the signal (open the relevant platform page / check the env var / inspect the log source)
  - either fixed, suppressed, or escalated into a real slice via /to-issues
  - this anomaly file deleted (or `triage: in-review` flipped) once resolved
---

## Signal

<one-paragraph description of the out-of-scope observation, with timestamps and IDs>

## Why it's out-of-scope for auto-alarm

<one-paragraph: this signal lives in a channel the runtime-log classifier doesn't watch — e.g. deploy state, build logs, env vars, log source availability. Auto-firing on it would either spam (every transient build retry) or miss (the bug only matters when correlated with other signals). Operator surface is the right escalation.>

## Operator action

1. <concrete first step, e.g. open the platform's run/job page>
2. <second step, e.g. compare x-request-id header to the recorded reqId>
3. <third step, e.g. file a real slice via /to-issues if confirmed>

## Out of scope

- Fixing the underlying issue is NOT this file's job — `/to-issues` will produce a proper slice if the operator confirms the signal is real.
```

NNNN derives from `ls <issues_dir>/anomaly-*.md | …`, default 1. Keep this count separate from `case-` and `auto-` (they're three independent monotonic series; don't share a counter — files are easy to filter by `case-*` / `auto-*` / `anomaly-*` prefix).

## Auto-issue template

When a new signature is detected, write `<issues_dir>/auto-NNNN-<slug>.md`:

```yaml
---
id: "auto-<NNNN>"
title: "AUTO: <first 60 chars of error msg>"
priority: P0
estimate_hours: 0.5
status: pending
owner: unassigned
blocked_by: []
blocks: []
mock_of: null
parallel: true
labels: [auto-detected, triage-needed, monitor-deploy]
triage: ready-for-agent
acceptance:
  - root cause identified (link to commit + line)
  - fix implemented in a code edit
  - regression test added (style left to the project's test runner)
  - .agent/bug-signatures.json updated with the proper memo (replacing "AUTO-DETECTED — needs triage")
---

## Why this issue exists

Auto-filed by `/monitor-deploy` skill at `<ISO ts>`. The signature did not match any known bug pattern in `.agent/bug-signatures.json`, so the skill couldn't dedupe it.

## First observed

- **reqId**: <reqId>
- **Route**: <route>
- **Status**: <status>
- **Source run/job id** (platform-specific): <id>
- **Source URL** (platform-specific): <url>

## Error snippet (first 240 chars of the offending line)

```
<line text>
```

## Full per-job log (all lines with this reqId from the tick)

<bulleted list of every line in the bucket, in chronological order>

## Suggested first step

<Project-specific pointer — e.g. `vercel logs <inspector_url> | grep '<reqId>'` for Vercel; `kubectl logs <pod> --since=1h | grep '<reqId>'` for k8s; `grep '<reqId>' logs/*.log` for file tails.> Then triage against existing signatures.

## Out of scope

- Fixing the root cause is the worker's job
- Updating `bug-signatures.json` to dedupe future hits is also the worker's job
```

The NNNN monotonic-derivation rule has been moved into step 5.3 (the workflow step that computes `next_n` from `ls <issues_dir>/auto-*.md`). The auto-issue template itself no longer needs to mention it.

## Hard rules

1. **Read-only against the running app.** This skill never touches the deployed app — only reads logs.
2. **Single project per invocation.** Don't try to monitor multiple projects in one tick.
3. **Idempotent.** Re-running with the same state must do nothing new. Specifically: the dedup MUST happen against `bug-signatures.json`, not against the rollup (which has no dedup semantics). Same for `auto-INDEX.md` and the three `*.md` counters (auto / case / anomaly) — each writes the next free id from `ls`, so a re-run with no new state finds no free ids and writes nothing.
4. **No subagents.** The skill is one assistant turn. If you need to file 5 auto-issues, file them all in one turn.
5. **Don't fire osascript more than once per signature.** First detection = one notification. Subsequent hits of the same signature increment `hit_count` silently — they DO write a new `case-NNNN-…` file (the audit trail demands it), but they don't fire osascript.
6. **Don't push origin.** Bug-signatures / state / rollup / auto-INDEX / case / anomaly files MAY be untracked (gitignored) OR committed — that's the user's call, not the skill's. If they're committed, fire `git add` + `git commit` per the user's `feedback-aggressive-push` posture, but only if there's a meaningful diff (don't commit empty no-op ticks).
7. **State file is the source of truth for "what was processed".** If the state file is corrupt or missing, default to `default_lookback` and rewrite it.
8. **Don't write noise.** Success cases that completed under `slow_threshold_ms` with no anomaly go to the rollup only — no `case-` file. A 50-runs-per-minute production would dump thousands of useless files into the issues dir; the audit value is the slow/failed ones.
9. **Case files are immutable.** Once a `case-NNNN-*.md` is written, it stays as-is. If the same reqId is observed again (e.g. an error retry), it gets a new `case-NNNN-*.md`, not an edit to the old one. The auto-INDEX row links both.
10. **Anomaly files are operator-resolved.** An anomaly doesn't auto-dispatch; it waits for a human to either delete it (false positive), flip `triage: in-review` (true positive, will be picked up by `/afk-agents` on next tick), or escalate via `/to-issues` (needs wave planning + mocks).

## State files (write these at end of first invocation if absent)

- `.agent/bug-signatures.json` — project-supplied known patterns. **Skill does NOT seed defaults.** Start empty or with the project's own known-bug list.
- `<state_file>` (default `.agent/monitor-state.json`) — `{ last_seen_ts, tick_count, total_alarms, new_signatures_this_session }`
- `<rollup_file>` (default `.agent/monitor-rollup.md`) — append-only summary, rotates at `rollup_max_lines`
- `<issues_dir>/auto-INDEX.md` — append-only ledger of every file the skill wrote (auto / case / anomaly)
- `<issues_dir>/auto-NNNN-*.md` — auto-issues for NEW signatures
- `<issues_dir>/case-NNNN-*.md` — case records for slow successes + known-bug hits
- `<issues_dir>/anomaly-NNNN-*.md` — anomaly records for out-of-scope findings (deploy-state errors, log source unreachable, etc.)

## Trigger phrases

`/monitor-deploy`, `/monitor-deploy <log-source> <lookback>`, "monitor the deploy", "tail logs", "watch for bugs", "alert me on new bugs", "deploy watcher", "background monitor", "watch the canary", "is anything broken on prod", "who's paged about the canary".

## Re-trigger cadence (recommended)

- `/loop 5m /monitor-deploy` — every 5 min. Within one typical request run-length so a new bug surfaces within one full job after it appears. Most ticks are no-ops (`jobs=0 new=0`).
- Pair with `/agent-status` for a human-readable rollup at the end of the day.

## Why this skill exists

Most platforms give you logs but not classification, dedup, or surfacing. `/loop` gives you a re-trigger cadence but not a target prompt. This skill is the target prompt: it's where the classification ruleset lives, where new signatures get fingerprinted, and where the macOS notification + auto-issue chain converges. The instrumentation that feeds it (per-request id tagging + structured heartbeat + phase timings) is project-specific — but the classifier, dedup, and three-file audit-trail contract (`auto` / `case` / `anomaly` in `<issues_dir>` + `auto-INDEX.md`) are reusable across any platform.

## Porting from a project-specific fork

If you're migrating from a project-specific version of `/monitor-deploy` (e.g. one that hardcoded Vercel + a specific reqId format), the diff is:

1. Replace hardcoded log-source calls with the `log_source: { type: ... }` config block.
2. Move the hardcoded reqId regex into `reqid_pattern`.
3. Move the hardcoded `bug-signatures.json` seed into `examples/bug-signatures.example.json` (or just commit the project's own seed).
4. Replace the hardcoded notification title with `notification.title`.
5. Move thresholds (`30000` ms, `15m` lookback, `500`-line rollup) into the config so per-project tuning is a 1-line YAML edit.

Everything else (the 5-step classification flow, the 3 templates, the 10 hard rules) is the same.

---
> Source: [liush2yuxjtu/monitor-deploy](https://github.com/liush2yuxjtu/monitor-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
