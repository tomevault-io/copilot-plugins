## herdkit

> Conventions every agent working in this repo (builder, resolver, coordinator, or a human) must

# AGENTS.md — herdkit engine conventions

Conventions every agent working in this repo (builder, resolver, coordinator, or a human) must
follow. This file is the portable, runtime-agnostic sibling of the auto-loaded `CLAUDE.md`: Claude
Code reads `CLAUDE.md` for free, but a non-Claude runtime (grok, codex, …) does not, so the engine
inlines these conventions into every builder's task spec and grounds grok's system prompt from this
file. Keep it short, factual, and current.

## The ship-a-change pipeline (code → PR → gate → merge)

1. **Build ONLY your change in your worktree.** Each item is built in an isolated git worktree; do
   not reach outside it.
2. **Verify your OWN surface, then open a PR.** Run `scripts/herd/healthcheck.sh "<worktree>" --light`
   (per-changed-file syntax) plus any test you added or changed, and get a clean pass. Fix any CODE
   errors; data/env warnings are fine. The whole-project heavy profile is DESCOPED for builders —
   the auto-merge watcher re-runs the full profile as the authoritative merge gate.
3. **The watcher owns the merge.** A builder never merges its own PR. The watcher merges ready PRs
   only after both gates go green (healthcheck + adversarial pre-merge review). If your change needs
   a manual step you cannot perform (a live smoke test, a UI/pane check), declare it in a
   `HUMAN-VERIFY:` block in the PR body — one step per line — which holds the PR for a human approve.

## Ownership boundaries

- **The tracker and `BACKLOG.md` are coordinator-owned.** Builders NEVER edit `BACKLOG.md` and never
  write the work tracker (a Linear/GitHub issue's state, labels, or assignee). A builder that mutates
  tracker state corrupts the queue. The coordinator owns ALL item states.
- **Never read or commit `.herd/secrets`.** Credentials never land in a committed or generated file.
  `DENY_PATHS` stays honored.

## Scratch A/B checkouts

- **To compare your change against a clean base, use a throwaway detached worktree, never `git
  stash`.** A `stash push <pathspec>` + `pop` can strand edits staged-but-reverted; a detached
  checkout is disposable and never touches your live worktree.
- **Name it `scratch-<anything>`** so the sweep's detached-scratch reaper recognizes it on sight
  instead of leaving it to age out:
  ```
  wt="$(mktemp -d)/scratch-ab"
  git worktree add -q --detach "$wt" HEAD
  trap 'git worktree remove --force "$wt" 2>/dev/null || true' EXIT
  ```
- **Pair every `git worktree add --detach` with a `trap … EXIT` remove**, so a crashed or
  interrupted session never strands it. If the trap doesn't fire, it still isn't lost debris: `herd
  sweep` reaps any detached, clean, zero-unique-commit worktree with nothing live inside it — not
  only ones matching a `scratch-*`/`tmp-*` name — right away outside `$WORKTREES_DIR`, or after a
  short age floor inside it. A worktree carrying commits that exist nowhere else is never reaped;
  it's flagged for a human instead.

## Design invariants for new behavior

- **Default-on, kill-switch-off.** New behavior lands with its config key defaulting **ON**. A fix,
  incident containment, or capability withheld behind a default-off flag is a fix nobody gets —
  "merged, default-off, no plan" is an incomplete feature, not a safe landing. The lever exists as
  a HARD no-op **kill-switch**: setting it off restores the prior behavior exactly, so an operator
  can disable a misbehaving change in one config write. This holds for **every** new lever — there
  is no "optional features may ship dormant" exception.
- **Soak is `ENGINE_TRACK`, not default-off.** The safety margin between a change landing and every
  consumer running it is the track split, not a dormant flag: `staging` follows main and gets new
  behavior immediately (it soaks there), `prod` gets it on `herd promote`. Default-off is never a
  substitute for that soak.
- **Byte-identical-when-off.** With the new lever off, output/argv/task-specs/generated files must be
  byte-for-byte identical to before your change. Tests assert this.
- **Fail-soft.** A missing OPTIONAL tool, file, or capability skips SILENTLY — it never produces a red
  row and never aborts a caller running under `set -euo pipefail`. Gate keys fail STRICT (fall back to
  the safest default and warn loudly); cosmetic keys fail soft to the documented default.

## Design invariants for load-bearing infrastructure (GH #964 lessons)

These invariants apply to any change touching retry loops, pane/marker lifecycle, launcher paths,
or health assessment. Violations are the root cause of retry storms and deadlock:

- **Bounded retries on every rail.** Every loop that retries on transient conditions (network timeouts,
  pane spawn failures, verdict collection) must have a NAMED, ENFORCED cap (`REFIX_MAX_ROUNDS`,
  `HEALTH_CONCURRENCY`, `SPAWN_CONCURRENCY`, etc.). A loop that can retry forever is a hanging
  vulnerability. Exhaustion logs `escaped retry cap` or similar, and escalates to a human.
- **Acquire/release pairing for panes and markers.** Every pane allocated (agent spawn, tab create,
  review collector start) must have a matching release (kill, close, reap). Every marker written
  (a lock file, an inflight record, a queued intent) must have a matching delete/clear path.
  Unpaired acquires leak state and cascade into roster-liveness wedges (the engine thinks resources
  are live when they are dead). Test this with `SANDBOX_FORCE_PANE_STALE=1` in sim scenarios.
- **Fault injection for launch paths.** Any path that starts a child process (agent spawn, review
  launch, resolver dispatch) must survive a **one-time failure on the first attempt** without
  wedging. Inject via `SANDBOX_FORCE_LAUNCH_FAIL=1` to prove a launch can fail, retry cleanly, and
  succeed on round 2. A launch that fails once and then silently gives up (no escalation, no retry)
  is load-bearing broken — you will ship undetected hangs.
- **Calm ≠ healthy — silence is not a signal.** A stuck loop can be perfectly silent (a blocked
  syscall, a hung network read, a pane that has exited but is not reaped). Distinguishing health
  from wedlock requires **explicit checks**: is the pane pid still alive (`ps`), is the output
  actively growing (tail -f), is a retry happening (journal grep), or is it just hung? `HEALTH_IDLE`
  thresholds and liveness probes are load-bearing. A monitor that only checks for errors (reds in
  the log, failed retries) and reports silence as green is an engine bug waiting to happen.

## Design invariants for liveness / adoption / reap changes (HERD-997)

These invariants apply to any change touching reviewer adoption, roster liveness checks, or
pane/slot reap logic. Violations are the root cause of adoption-of-corpses and done-slot squatting:

- **Three-legged liveness-fix convention.** Any liveness, adoption, or reap change MUST handle all
  three terminal states and prove each with a dedicated test:
  1. **`working`** (agent/pane is alive and actively running) → **adopt or leave in place** — never
     interrupt a live reviewer.
  2. **`done` + current sha** (pinned sha matches the PR's live HEAD) → **collect** — the agent
     finished its assignment; retire the slot cleanly (read its verdict, close the pane).
  3. **`done` + stale sha** (pinned sha differs from live HEAD, or HEAD moved on) → **reap** — the
     slot is a corpse squatting the builder tab; remove it unconditionally so it cannot be re-adopted
     and block the next placement.

  A change that handles only one or two legs ships a liveness bug waiting for the third state to
  surface in production. Name the test for each leg explicitly in the PR body.

- **DONE-means-a-real-transition.** `DONE` is reserved for a confirmed, observable state change —
  the agent finished the task and filed a verdict. Never label a dropped write, an unreachable branch,
  or a skipped step `DONE`. Each of those states gets its own truthful token (`DROPPED`, `ABANDONED`,
  `UNREACHABLE`) so log readers can distinguish "it finished" from "we gave up" or "that path is dead
  code." An unreachable path that emits `DONE` reads as success in every downstream consumer —
  verdict collectors, the watcher's round budget, and `herd why` alike.

## Commits & attribution

- **NO AI co-author trailer on commits.** Never add `Co-Authored-By: Claude …` or a
  `Generated with Claude` line. (`ATTRIBUTION_POLICY=no-ai-coauthor` enforces this at the gate.)

## Testing discipline

- **Run `scripts/herd/healthcheck.sh` before every PR** (see the pipeline above).
- **Sim-first for load-bearing changes.** Any change to gate / merge / concurrency / limit / pane
  behavior must be proven with a simulation, not only unit asserts. Use the scenarios under
  `scripts/herd/sim/` (e.g. `sandbox-concurrency-scenario.sh`, `retirement-invariant-sim.sh`).
- **Prove your lever both ways.** Add a test asserting behavior when ON and byte-identical output
  when OFF.
- **Hermetic fixtures vs production defaults.** A default-on flip (HERD-879) or loader env-scrub
  (HERD-885 F1) reds *older fixtures*, not the new feature. The **hermetic-fixtures** agent owns
  the checklist; the short form:
  - Spawn/enqueue without a tracker ref: `export TRACKED_SPAWNS=off` (re-arm in the case that
    asserts the gate). Do not invent a tracker id.
  - Pin `PROJECT_ROOT` / `WORKTREES_DIR` / `WORKSPACE_NAME` **in the fixture `.herd/config`**, not
    only env — the loader unsets those before sourcing a real config file.
  - Tests that assert engine `:=` defaults: `HERD_CONFIG_FILE` to a missing path. `env -i` still
    walk-up-loads dogfood `.herd/config`; a laptop `config.local` overlay can hide that from CI.
  - `herd notes` / `bin/herd` children: same config pins **and** child env (`HERMETIC_TEST`,
    `HERD_JOURNAL_HERMETIC`, `JOURNAL_FILE`). Unpinned `WORKTREES_DIR` binds the live ledger.
  - `value_shape` is `numeric`/`free`/`glob`/empty or pipe-literals — never prose (`0..1 float`).
  - Touching `templates/capabilities.tsv` wide-blasts the full curated suite. Run the **named**
    CI failures; a journal row is not always the red.

## Config key naming

Name config keys after the **behavior** they gate, not the **incident** that motivated them.

- **Good:** `REVIEW_PLACEMENT_GUARD` — names what the key guards (review placement)
- **Bad:** `REVIEW_PLACEMENT_STORM` — names the incident that triggered the feature

An incident-named key reads as nonsense to anyone who joined after the incident, silently
erodes the config surface's legibility, and couples the doc permanently to a moment in time.
A behavior-named key is self-documenting forever.

The pattern: identify the protected invariant or behavioral change, then name the key after
_that_.  If you find yourself typing the name of an alert, a postmortem, or a GH issue title
into a key name — stop and rename it.

## Config key lifecycle (HERD-916)

Every lever has a **lifecycle stage** recorded in `templates/capabilities.tsv`'s
`lifecycle_stage` column. New levers land at `default-on:<YYYY-MM-DD>` (default-on is the landing
stage — see *Design invariants for new behavior*); `introduced-off` is a **legacy** stage kept only
for levers that landed dormant before the 2026-08-27 convention change, never the target for new work:

| Stage | Meaning |
|---|---|
| `default-on:<YYYY-MM-DD>` | Default is ON since DATE (the landing stage for new levers); candidate to delete after 180 days |
| `introduced-off` | **Legacy** — landed dormant before 2026-08-27; awaiting its default-on flip. Not for new levers |
| `proven:<YYYY-MM-DD>` | A legacy `introduced-off` lever proven in production since DATE; candidate for `default-on` after 90 days |
| `removed` | Lever gone; row kept for history |

`herd doctor --posture` surfaces **CANDIDATE-TO-FLIP** (proven ≥ 90 days) and
**CANDIDATE-TO-DELETE** (default-on ≥ 180 days) advisories. Advisory only — nothing
auto-flips.  The operator decides; when a default flips, `herd config sync`'s upgrade note
and the docs move in the same PR.

Thresholds are overridable via `LEVER_PROVEN_DAYS` / `LEVER_DEFAULT_ON_DAYS` (env, not
`.herd/config` keys).

## Orienting fast

A deterministic map of the engine tree is committed at `docs/codemap.md` (module roles,
who-sources-whom, config-key → consumer wiring; regenerate with `herd codemap`) and a function-level
def→caller index at `docs/symbol-index.md` (`herd symbol-index`). Read them FIRST to skip
re-exploring the tree.

---
> Source: [briankeegan1/herdkit](https://github.com/briankeegan1/herdkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
