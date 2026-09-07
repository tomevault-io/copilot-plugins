## harbor-rl

> validates a script's shape for you, so four rules hold, enforced by

# harbor

Plugin for setting up Python simulation repos via uv and authoring RL tasks end-to-end.

## Hard constraints (apply to ALL tasks)

1. Generated `install.md` / `history.md` / `benchmark.md` MUST be English-only — regardless of chat language.
2. **Dispatch depth ≤ 2** (main → orchestrator → worker). Exactly ONE agent carries the `Agent` tool: `reward-tuning-agent`, which dispatches `reward-candidate-agent`. Every other agent is a leaf and does its delegated work itself; the main thread orchestrates the rest (`dependency-generator` → `benchmark-generator` → …). Enforced by `tests/contract/test_agent_depth.py`. Nested dispatch needs **Claude Code ≥ 2.1.219** — `/harbor:reward-tune` asserts it in pre-flight and stops rather than degrading.
3. **All plugin-generated files live under `<repo>/harbor/`** — except `scripts/_<family>_env.py` / `scripts/run_random.py` / `scripts/render_random.py` (user-facing smoke entry points). Each generator agent writes its receipts + metadata into **its own subdir**: `harbor/dependency-generator/{setup_uv.sh, probe.json, install_plan.json, install.md}`, `harbor/benchmark-generator/{benchmark-spec.json, task_overview.md, .task_list.json, history.md, benchmark.md}`, `harbor/rl-integration-generator/{rl-suite-spec.json, rl-integration.md, history.md}`. The shared RL training tree stays at the top level: training scripts at `<repo>/harbor/scripts/rl/`, configs at `<repo>/harbor/configs/rl/`, training output at `<repo>/harbor/outputs/`, the DataLogger at `<repo>/harbor/utils/data_logger.py`; the create-task workspace at `<repo>/harbor/create-task/`. There is NO shared `run-log/` folder — each agent's per-run process log is the `history.md` inside its own subdir. The folder name is `harbor/` (no dot) so it doubles as a valid Python package — imports like `from utils.data_logger import DataLogger` resolve against `<repo>/harbor/` after `sys.path.insert(0, HARBOR_ROOT)`.
4. Code style across main thread AND all subagents:
   - **Think before coding** — state assumptions explicitly; if uncertain, ask. Don't pick silently between alternatives.
   - **Simplicity first** — minimum code that solves the problem; no speculative features, abstractions, configurability, or error handling for impossible scenarios.
   - **Surgical changes** — touch only what the task requires; don't "improve" adjacent code, refactor things that aren't broken, or remove pre-existing dead code unless asked.
   - **Goal-driven execution** — define verifiable success criteria up front; loop until the verification check passes. Weak criteria like "make it work" are not acceptable.

---

## 6-layer mental model

The harness is structured as six layers with different cardinality, lifecycle, and mutability. Use this map when deciding where a new module belongs.

```
L1   AGENT (intelligence)         — Claude itself; not in code
L2   ENTRY POINTS                 — User-facing surfaces. commands/<name>.md and
                                    skills/<name>/SKILL.md are the SAME mechanism upstream
                                    (custom commands were merged into skills); both create
                                    /harbor:<name> and support the same frontmatter. Skills
                                    add only a per-entry-point directory for supporting files.
                                    harbor uses commands/ throughout: its supporting files
                                    (knowledge/references/, knowledge/templates/, knowledge/experiences/) are shared across
                                    entry points, not owned by one.
L3   SUBAGENTS (roles)            — agents/<name>.md   (fresh context, isolated agent loop)
L4   TOOLS (deterministic)        — scripts/<owner>/*.py + Bash + Read/Write/Edit
L5   SHARED KNOWLEDGE (read-only) — knowledge/templates/, knowledge/references/, knowledge/experiences/
L6a  WORKSPACE PROCESS LOGS       — <repo>/harbor/<agent>/history.md   (per-run, append-only, inside each agent's subdir)
L6b  WORKSPACE RECEIPTS           — <repo>/harbor/<agent>/{install,history,benchmark,rl-integration}.md  (end-of-run user summary, in each agent's subdir)
```

Decision rules when adding a new module:

```
Q1: Does the user invoke it directly in chat?    → L2 entry point
Q2: Multi-step reasoning + decisions?            → L3 subagent
Q3: Single deterministic input → output?         → L4 tool
Q4: Read-only doc / data?
    Q4.1: Cross-repo shared?                     → L5 shared knowledge
    Q4.2: Per-run process record?                → L6a process log
    Q4.3: End-of-run user-facing summary?        → L6b receipt
```

**Invocation boundary (L2).** By default an entry point is both user- and model-invocable. Exactly ONE command is gated with `disable-model-invocation: true` — **`reset-workspace`**, the only irreversible operation (`git reset --hard` + `git clean -fdx`). Everything else stays model-invocable **on purpose**: harbor's commands compose (`rl-sweep` → `rl-run`, `task-create` → `reward-tune`, `test` → the whole chain), and a gated command cannot be invoked by another command at all — gating a building block silently breaks every chain that calls it. When a chain needs gated behavior, it **Reads the command body and executes it** rather than slash-invoking (see `test.md` stage 11 and `task-create.md` §6). Enforced by `tests/contract/test_invocation.py`: the gated set is exactly `GATED`, nothing else carries the flag, no gated command is slash-invoked internally, and all frontmatter is valid YAML (a malformed block silently drops every field, gating included).

L2 vs L3 are **not the same axis**:
- L2 asks "how does the user wake it up" (slash invocation)
- L3 asks "how is the context isolated" (fresh `messages=[]`, independent loop)
- Skills can dispatch subagents internally; subagents don't need a slash entry. The two are independent.

---

## Where things live (post-refactor)

### L2 — Entry points

Commands are grouped by area via filename prefix (Claude Code commands have no true subdir namespace — the prefix IS the group). Groups: `env-*` · task (`task-*` + `probe-*`) · `reward-*` · `rl-*` · top-level utilities.

- `commands/help.md` — `/harbor:help` plugin overview

**env — environment setup**
- `commands/env-install-uv.md` — `/harbor:env-install-uv [path]` — uv-only env setup; renders `<repo>/harbor/dependency-generator/setup_uv.sh` and creates `<repo>/.venv/` (dispatches the `dependency-generator` subagent)

**task — author / probe / inspect tasks**
- `commands/probe-benchmark.md` — `/harbor:probe-benchmark [repo=<path>] [canonical_task=<id>]` — author `<repo>/harbor/create-task/task-implementation.md` (Step 3.7 of `benchmark-generator`, extracted so it can be re-run standalone or delegated from the agent)
- `commands/probe-task.md` — `/harbor:probe-task task=<id> [repo=<path>] [output=<path>]` — emit a per-task `<task-slug>-implementation.md` capturing every design choice (scene / actions / reset / termination / observation / reward / DR) with verbatim code. **Runs in a subagent** (context-saving). Feed back into `/harbor:task-create from=<path>` to clone the task identically into another benchmark.
- `commands/task-create.md` — `/harbor:task-create name=<TaskID> (description="..." | from=<spec.md>) [sections=<list>] [assets=<paths>]` — author a NEW task in the current benchmark repo, or reproduce one from a `/harbor:probe-task` spec. Pre-flight checks the `dependency-generator` → `benchmark-generator` → `rl-integration-generator` chain in sequence and dispatches any missing stage first (rl-integration defaults to `custom_torch` unless the user specifies an algorithm source). Then runs `task-generator` (§1–§5, per-section smoke gates) → the `/harbor:reward-tune` loop (§6 — dispatches `reward-tuning-agent`, validated by actual training until success_rate ≥ threshold in ALL modes; reproduce mode seeds iter 0 with the spec's reward pasted verbatim) → `dr-generator` (§7, **opt-in**: skipped unless the user explicitly requests DR). The main agent is **orchestrator-only** — it selects the design base and passes it down, but makes no §1–§7 design decisions (those belong to the authoring subagents).
- `commands/task-list.md` — `/harbor:task-list` — list/inspect tasks in a benchmark; reads the cwd-local `harbor/benchmark-generator/benchmark-spec.json`
- `commands/task-clone.md` — `/harbor:task-clone op=create source=<TaskID> dest=<TaskID> [info_out=<path>] | op=delete dest=<TaskID>` — clone a task into an isolated, independently-editable copy registered under a new suffixed gym id (`-rewarditer<NNN>` before `-vN`); `create` dispatches the `task-cloner` subagent, `delete` removes the clone. A general isolation primitive: A/B variants, cross-benchmark migration, and `/harbor:reward-tune`'s parallel candidates (which call `clone_task.py` directly rather than slash-invoking this command)

**reward — reward engineering**
- `commands/reward-tune.md` — `/harbor:reward-tune task=<id> [algorithm=<algo>] [pool_size=N] [gpus=N] [mode=local|cluster] [on_success=cancel|drain] [success_threshold=0.5] [timesteps_per_iter=N]` — ASYNC fixed-pool reward tuning. A **thin orchestrator**: the main agent does pre-flight (incl. the ≥ 2.1.219 nesting assert) + (standalone) base selection + every user-facing question, then dispatches **`reward-tuning-agent`** (DESIGN + DECIDE), which dispatches one **`reward-candidate-agent`** per candidate (IMPLEMENT + train/render + SCORE). Each candidate is a bounded §1–§5 task delta PLUS a complete reward, so the search covers both. Keeps `pool_size` candidates in flight — capped by `gpus` in local mode — loops until `success_rate ≥ success_threshold`, cancels in-flight, then PROMOTES the winning design onto the source task and re-verifies it with that winner's own smoke set. Isolation follows the effective pool: sequential over a `base/` snapshot at 1, one slot clone per candidate above 1. Findings shared via `<task_dir>/memories.jsonl`.
- `commands/reward-add-log.md` — `/harbor:reward-add-log` adds a per-term reward-visibility wrapper to `scripts/_<family>_env.py` (does NOT modify env reward); the sanity-check smoke asserts `composer(info["detailed_reward"].values()) == env_reward` per step, where composer ∈ {"sum","product"} is per-task. Assets at `scripts/reward-add-log/` + `knowledge/templates/reward-add-log/`.

**rl — train / eval / policy**
- `commands/rl-run.md` — `/harbor:rl-run task=<id> algorithm=<algo> [k=v ...]` — single-trial training; wraps `harbor/scripts/rl/<impl>/train.py` against `<repo>/.venv/bin/python`
- `commands/rl-eval.md` — `/harbor:rl-eval checkpoint=<path> [k=v ...]` — single-checkpoint eval; writes `metrics.json` next to checkpoint
- `commands/rl-render.md` — `/harbor:rl-render checkpoint=<path> [k=v ...]` — render a checkpoint to MP4 with inference-moved + frame-difference sanity checks
- `commands/rl-visualize.md` — `/harbor:rl-visualize checkpoint=<path> [k=v ...]` — open headed GLFW viewer; requires `$DISPLAY`
- `commands/rl-sweep.md` — `/harbor:rl-sweep task=<list> algorithm=<list> [k=v1,v2,...]` — Cartesian-product sweep; one sub-agent per trial; results under `harbor/rl_experiments/sweeps/<sweep_id>/`
- `commands/rl-tune.md` — `/harbor:rl-tune task=<list> algorithm=<list> [mode=local|cluster]` — Cartesian-product grid TUNING (open-ended hyperparameter loop); one rl-tuning-agent subagent per cell; tune-level history.md + final cross-cell summary under `harbor/rl_experiments/tunes/<tune_id>/`. One-time scaffolding lives in the `rl-integration-generator` subagent (dispatched directly).
- `commands/rl-add-trick.md` — `/harbor:rl-add-trick <trick> [algorithm=<algo>]` — apply an RL training trick (e.g. `obs_rms_jax`, `reward_norm_jax`) to a chosen algorithm config in-place (reads the `knowledge/templates/rl-tricks/` library)
- `commands/rl-list-tricks.md` — `/harbor:rl-list-tricks` — list available RL training tricks with descriptions + applicability (read-only)
- `commands/rl-add-log.md` — `/harbor:rl-add-log` — canonical metric-key contract (PPO/SAC/TD3 + per-reward-term + SB3 remap) for ALL `harbor/scripts/rl/<impl>/` algorithms; the binding reference `rl-integration-generator` follows when authoring/patching algorithm training code

**utilities**
- `commands/plot.md` — `/harbor:plot spec=<yaml>` — multi-panel mean±std W&B learning curves grouped by task × baseline
- `commands/wandb-setup.md` — `/harbor:wandb-setup` — inspect / re-login / switch the host's W&B account
- `commands/reset-workspace.md` — `/harbor:reset-workspace repo=<path> [clean_inbenchmark_tasks=true|false]` — **destructive**: remove ALL plugin output from a benchmark repo (`harbor/`, `.venv/`, `scripts/` carve-outs, caches) and (default) `git reset --hard` + `git clean -fdx` it back to its original cloned HEAD. Runs in a subagent with a dry-run + confirm gate and a git-based smoke (incl. hidden / ignored files) that must fully pass before reporting success
- `commands/test.md` — `/harbor:test [layers=1,2,3] [repo=<path>] [task=<id>] [from_spec=<path>]` — plugin test runner. L1 (contract) + L2 (unit) are deterministic `pytest tests/{contract,unit}` (main thread). L3 is an e2e pipeline (subagent) driving the task-create→train→reset chain module-by-module on an isolated clean benchmark **worktree**, using a benchmark-agnostic stack-two-cube fixture (create mode) with §6 bounded to one iteration (`success_threshold=0`). Resumable Docker-layer style via `scripts/test/pipeline.py` (per-module fingerprints → re-run only changed/failed stages onward); append-only `history.md`; fail-fast with suggested fix; skips dr-generator + headless modules
- `commands/update-experience.md` — `/harbor:update-experience target=<name> (experience="..." | file=<path>)` — append a numbered bullet to an agent ledger (`reward-tuning-agent`/`task-generator`/`dr-generator`/`rl-tuning-agent`; hand-written bullets capped at 5 lines), OR file a `/harbor:probe-task` spec into the right `knowledge/experiences/task-library/` embodiment folder (classify manipulation vs humanoid/quadrupedal locomotion; short `<task>-<repo>.md` name, `-vN` on collision)

### L3 — Subagents (heavy, multi-step; main thread dispatches; depth ≤ 2)

- `agents/dependency-generator.md` — entry point for any "set up env for \<repo\>" task; renders `<repo>/harbor/dependency-generator/setup_uv.sh`, creates `<repo>/.venv/`, runs the import smoke
- `agents/benchmark-generator.md` — env-sanity layer: random rollout + render-to-MP4 + 2-tier smoke (L1 random / L2 render). Always treats the repo as RL — no IL detection. Does NOT generate train/eval scripts (rl-integration-generator owns those).
- `agents/rl-integration-generator.md` — RL experiment scaffold: configs, train/eval/render/visualize scripts, algorithm adapter, smoke per algorithm
- `agents/rl-tuning-agent.md` — algorithm-by-algorithm hyperparameter tuning loop (train→eval→render→analyze→suggest)
- `agents/task-generator.md` — authors §1–§5 of a new task (register/scene · actions · reset · goal+termination · observation) with per-section smokes plus an actuator-tracking check (S2.5) and a render-stability + visual check (S6); looping each smoke until it passes (escalates on two attempts that fail to move the measured quantity, or a 10-attempt backstop). Reads `<repo>/harbor/create-task/task-implementation.md`. Dispatched only by `/harbor:task-create`.
- `agents/task-cloner.md` — clones a task's editable surface (env_cfg + the mdp modules the requested `surface` covers) into dest-named copies, rewires imports, registers `<dest>` (suffix before `-vN`), runs the clone smokes (build + rollout + per-term-logging), writes a delete manifest. Dispatched by `/harbor:task-clone op=create`; `reward-tune` calls the underlying `clone_task.py` directly for its slot clones. Never edits source files.
- `agents/reward-tuning-agent.md` — the §6 **designer**: reads `task-history.md`'s §1–§5 Analysis first (§1 failure modes · §2 what the action space can express · §3 the start layout · §4 the subgoal decomposition that IS the term ladder, plus the degenerate states that are the reward-hacking surface · §5 what a term may key on), then DESIGN (each candidate = a bounded §1–§5 task delta + a complete reward; B1 — adapt-first, magnitude budget, concrete weights/gates/composer, in-flight-aware distinctness) + DECIDE (best-so-far, convergence, refill, PROMOTE the winning design onto the source task and re-verify with that winner's smoke set). Runs an async fixed pool of `pool_size` candidates — capped by `gpus` in local mode — until `success_rate ≥ threshold`. **The one agent with the `Agent` tool** (depth ≤ 2): it dispatches `reward-candidate-agent` per candidate and never writes reward code itself, which is what keeps implementation noise out of the design context. Sole writer of every shared file; per-term logging is the `/harbor:reward-add-log` flow run in-line. Dispatched by `/harbor:reward-tune` (standalone) and `/harbor:task-create` (§6). Checkpoints `tune-state.json` every iteration (resume-safe); returns `needs_decision` for the caller to put to the user.
- `agents/reward-candidate-agent.md` — ONE candidate end to end: IMPLEMENT its §1–§5 task delta + reward into its own task (slot clone, or the source when sequential), authoring each touched section from the same `knowledge/references/task-generator/s<N>-*.md` file `task-generator` used → run that section's smokes plus S6, reading every result out of the `<smoke>.verdict.json` the shared `_verdict.py` recorder writes, and `Read`ing the visual passes' keyframes → train + render (local bg or SLURM, sentinel watchdog, optional mid-run early-stop monitor) → SCORE via `scripts/reward-tuning-agent/score_iter.py` + rendered frames → verdict JSON. Implements only — never designs, never reweights to pass a smoke. Reports `task_smoke_failed` and `reward_smoke_failed` distinctly, since they lead to opposite next moves. Leaf agent (no `Agent` tool); dispatched only by `reward-tuning-agent`.
- `agents/dr-generator.md` — authors §7 (domain randomization) across 3 groups (robot · object · observation-noise) at the placeholder, once-per-episode-per-env (`mode="reset"`). Discovers available terms per group and wires EVERY available term by default (comprehensive, not minimal — hard constraint; un-wired terms need a logged reason) (modes: multiplicative/additive/direct for groups 1–2 default `(0.9,1.1)`; uniform/gaussian for obs noise default σ=0.01), runs the §7 smoke (exact value read-back at num_envs=16 + after-reset re-check), and writes a handoff at `harbor/create-task/<slug>/handoff-dr-generator.md`. Final agent in the `/create-task` chain; `skipped` is a valid success when no DR is requested and the canonical example has none.

### L4 — Tools (deterministic CLIs)

harbor's tools are plain CLI scripts invoked with Bash — deliberately, not MCP. The heavy
operations are backgrounded GPU jobs whose state must survive a killed agent or a resumed
session, which is what files + exit codes give you and an in-process server does not. Nothing
validates a script's shape for you, so four rules hold, enforced by
`tests/contract/test_script_conventions.py`:

1. **`argparse`**, so `--help` works and an agent can discover the interface.
2. **JSON to stdout** whenever an agent parses the output.
3. **exit 0 = the tool ran; non-zero = bad input, it could not run at all.**
4. **Emit a verdict object even when the answer is "it failed."**

Rules 3–4 are the subtle pair. `score_iter.py` is the worked example: no `metrics.jsonl` →
exit 0 with `{"success_rate": null, "gate": "no_metrics"}` (the tool worked; the answer is
"ungradable"), while a missing `design.json` exits non-zero (the caller passed something
broken). Backwards, and an ungradable candidate reads as a broken script.

**The tool layer is host-independent.** A script finds its own tree from `__file__`, never
from `CLAUDE_PLUGIN_ROOT` — that env var is ambient state a stale or foreign value would
silently win, pointing a script at another plugin's tree, and nothing outside Claude Code
sets it. Callers needing a different root pass an explicit `--plugin-root`. This is what lets
`scripts/` run under Codex, from a bare shell, or in CI unchanged. (The `${CLAUDE_PLUGIN_ROOT}`
string still appears in markdown and docstrings — that is the L2/L3 layer's path syntax, which
Claude Code expands; it is the runtime lookup that is banned.)

**Coverage is a ratchet.** Every script needs a unit test; `UNTESTED_DEBT` in that contract
test may only shrink, and scripts that genuinely cannot run in CI (need a sim, a venv, or the
network) sit in `UNTESTABLE` with a stated reason.

```
scripts/
  common/                    run_with_sentinel.sh  (completion is the ARTIFACT, not the
                             exit code: poll for a sentinel, grace, then tear down the
                             process group — a GPU-sim trainer killed AFTER its checkpoint
                             is a success)
                             resolve_suite.py  (canonical rl-suite-spec.json reader: slug / scripts_dir / parallel / config_name — single source so the key path can't drift across callers)
  test/                      pipeline.py  (/harbor:test L3 stage engine: Docker-layer fingerprint cache → plan/mark/stages for resumable module-by-module e2e)
  dependency-generator/      render_uv.py, smoke_uv.py
  benchmark-generator/ capture_spec.py, list_tasks.py, render_task_overview.py
  rl-integration-generator/ render_rl_suite.py, render_data_logger.py, discover_rl_tasks.py,
                      check_trial_artifacts.py (the deterministic half of the rl-suite
                      smoke: T4 curves + T5 TB/jsonl, and trial-dir resolution from a log),
                      discover_algorithms.py, validate_rl_suite.py
  reward-tuning-agent/ curve_health.py  (compact health snapshot of a LIVE reward-tune
                      metrics.jsonl → evidence + advisory concern flags for the candidate's
                      mid-run monitor / confident early-stop; gathers, never kills)
                      score_iter.py  (the ONE success_rate formula: metrics.jsonl +
                      design.json → the candidate's verdict.json; an ungradable run
                      returns success_rate=null + a gate reason, never a fabricated score)
                      _metrics.py  (shared long/wide metrics.jsonl reader for both)
  task-generator/     coacd_decompose.py  (mesh -> <=32 CoACD parts + a .coacd.json manifest;
                      --inspect reports an existing USD/MJCF's collision provenance and admits
                      when a binary usdc is unreadable rather than calling it clean)
                      build_coacd_usd.py  (the middle of that pipeline: authors ONE convexHull
                      collider prim per part. Merging the parts into a single mesh instead
                      passes every provenance check while PhysX hulls the whole thing)
                      check_task_history.py  (the gate task-generator passes before it
                      returns: every analysis term answered, every table cell filled, and
                      every claimed verdict diffed against the smoke's own
                      <smoke>.verdict.json — the transcription stops being load-bearing.
                      --checklist-out also renders test-checklist.md, the checks that
                      ACTUALLY ran, which is task-specific: §4 emits one C<i>/V<i> pair per
                      predicate the design implemented. --analysis-out renders task-analysis.md,
                      the design-rationale half with Validations stripped — what §6's designer
                      and every reward candidate read instead of the full 72KB history)
  rl-run/             check_reward_logger.py
  rl-tricks/          apply_trick.py, list_tricks.py
  task-cloner/        clone_task.py  (deterministic same-repo clone for /harbor:task-clone:
                      discover source's register site + cfg (repo grep, no sim) → copy
                      editable surface → rewire imports → mirror gym.register (suffix before
                      -vN) → manifest; op=delete reverses it. Encodes the deterministic
                      clone-contract checks; SC1/SC3/SC4 sim smokes stay with the agent)
  reward-add-log/     sanity_check.py, sanity_check_isaaclab.py
  plot/               render_plot.py
  install/            install_prerequisites.sh, install_uv.sh
```

### L5 — Shared knowledge (read-only)

All three L5 kinds live under one root — `knowledge/{templates,references,experiences}/` — because they are one thing in this model: read-only material an agent loads on demand. Component dirs (`commands/`, `agents/`, `hooks/`, `scripts/`) stay at the plugin root, where Claude Code scans for them by default.

```
knowledge/templates/
  dependency-generator/         install.md.template
  benchmark-generator/   benchmark.md, history.md, task_overview.md,
                         task-implementation.md (read by /harbor:task-create),
                         scripts/{run_random, render_random}.py.template
  rl-integration-generator/  per-source subtrees (renderer picks one):
                               stable_baseline3/scripts/{train,eval,render,env_wrapper}.py.template
                               custom_torch/scripts/{train,eval,render,env_wrapper}.py.template
                               custom_torch/{algo,replay,models,utils}/*.py.template (~14 self-contained algo files)
                               custom_jax/scripts/{train,eval,render,env_wrapper,visualize}.py.template +
                                 custom_jax/{algo,replay,models,utils}/*.py.template (JAX mirror of custom_torch)
                               local_implementation/scripts/{train,eval,render,env_wrapper}.py.template (shims)
                             shared (top-level):
                               configs/{ppo,sac,td3}{,.parallel}.yaml.template (unified schema both sources read)
                               configs/suite.yaml.template
                               rl-suite-spec.json.template (carries algorithm_slug + scripts_dir)
                               rl-integration.md.template (Layer 6b user receipt: train / eval / render / override-hparams)
                               tune.py.template (top-level cross-impl tuning entry)
                               data_logger.py.template (rendered to <repo>/harbor/utils/data_logger.py by render_data_logger.py)
  rl-tuning-agent/       tuning-history.md.template (per-cell ledger)
  rl-tune/               history.md.template (tune-level ledger written by /harbor:rl-tune)
  reward-tune/           history.md.template (tune-level ledger written by /harbor:reward-tune)
  task-generator/        task-history.md.template (the §1..§5 + S6-gate process-log scaffold rendered at
                           agent entry — one block per section, each with an Analysis and a
                           Validations subsection, filled in place by Phase A / Phase B);
                         smokes/_verdict.py.template (shared recorder every smoke imports;
                           writes <smoke>.verdict.json eagerly so a smoke's result reaches the
                           history as machine truth rather than agent recollection);
                         smokes/smoke_s{1..5}.py.template (per-section behavioral smokes,
                           num_envs=2 for gpu-sim; S1 runs C1..C8 incl. collision-solidity,
                           articulation-limit and collision-provenance checks, S3 runs C1..C5,
                           S4 runs G1 + one
                           C<i>/V<i> pair per implemented predicate) + smoke_s2_5.py.template
                           (actuator tracking error → physics-param sanity, runs after S2) +
                           smoke_s3_render.py.template (§3 visual pass: one frame per reset →
                           the agent judges the layout) + smoke_s6_render.py.template
                           (random-rollout render → scene-stability asserts + keyframe PNGs;
                           MP4 to <task_dir>/) + smoke_success_visualize.py.template (headed,
                           user-run success-scenario viewer; NOT run in regression — the
                           success predicate itself is one of S4's C<i>/V<i> pairs) —
                           all rendered to <task_dir>/smokes/ then run in .venv;
                         action_terms/ema_delta_joint_pos{,_cfg}.py.template (custom
                           EMACumulativeRelativeJointPositionAction — rendered into
                           <task>/mdp/ when §2 mode == ema_delta_joint_pos);
                         action_terms/ema_delta_ee_pose{,_cfg}.py.template (custom
                           EMACumulativeDeltaPoseAction — task-space analog;
                           rendered when §2 mode == ema_delta_ee_pose)
  reward-tuning-agent/   smokes/smoke_s6.py.template (reward finite + non-constant +
                           composer assertion via info["detailed_reward"]; builds the env
                           through scripts/_isaaclab_env.py so it tests the CANDIDATE's
                           reward and passthrough is a hard fail),
  task-cloner/           smokes/smoke_clone.py.template (cloned task builds + rolls out
                           with finite reward + reports per-term-logging inheritance;
                           one AppLauncher covering SC1/SC3/SC4)
  dr-generator/          smokes/smoke_s7.py.template (per-term exact value read-back at
                           num_envs=16 — point-interval range → prop == default modified by k,
                           re-checked after reset; obs-noise terms checked vs paired no-noise cfg)
  reward-add-log/        reward_terms_block.py.template (Path A scalar wrapper),
                         isaaclab_env_helper.py.template (Path B IsaacLab helper)
  rl-tricks/             <trick>/{manifest.yaml, patches.yaml, smoke.py, edits/*} — trick library read
                         by /harbor:rl-add-trick (obs_rms_jax, obs_rms_torch, reward_norm_jax,
                         value_clip_torch, value_norm_torch, distributional_critic_torch)
  rl-sweep/              launch.sh{,.isaaclab}.template (SLURM trial launchers — shared by
                         /harbor:rl-sweep and reward-tune's cluster mode, which renders one
                         trial per candidate so both inherit the site's proxy + WANDB_API_KEY)
  plot/                  spec.example.yaml (example /harbor:plot spec)

knowledge/references/
  task-library-search.md  cross-cutting: search the task-library and return the single most-relevant prior task BEFORE designing (read by /harbor:task-create Step 1.5, /harbor:reward-tune main agent)
  adapt-first.md          cross-cutting: how the authoring agents build from the selected base — read the ledger, port everything / change only overrides, document the delta (read by task-generator, reward-tune main agent, dr-generator)
  common/agent-conventions.md  cross-cutting: shared conventions (smoke pass-criterion, diagnose-and-retry, process-log discipline, English-only / no-nested-dispatch) for the authoring subagents — each agent's body overrides the generic shape with its own specifics
  dependency-generator/         decision-protocol, install-plan-schema
  benchmark-generator/   smoke-test-contract,
                         receipt-generation, case-studies,
                         task-implementation-contract (rules for the
                           /harbor:task-create guide)
  rl-integration-generator/ rl-suite-spec (schema for harbor/rl-integration-generator/rl-suite-spec.json + benchmark-spec rl extension)
  rl-tuning-agent/       tuning-instruction (procedure + 4 hard constraints)
  task-generator/        s1-scene · s2-actions · s3-reset · s4-termination · s5-observation ·
                           s6-render — ONE file per task section, each with the same six
                           headings (what it authors / decisions / API pointers / its smoke /
                           failure→diagnosis→fix / traps). Cross-cutting despite the folder
                           name: read by task-generator (the sections it authors) AND
                           reward-candidate-agent (the sections its task_changes touch) — both
                           load only the files they need. Plus isaaclab-code-reference (action /
                           scene / obs / termination / command APIs the section files point
                           into). README.md states the filing rule for new detail.
  reward-tuning-agent/   isaaclab-reward-reference (composer-by-family, RewTerm idiom,
                           common mdp.* building blocks, weight conventions),
                         smoke-contract (what S6 verifies + substitutions),
                         candidate-contract (designer↔candidate request/verdict schema,
                           the boundary rule, disjoint write scopes, status semantics)
  task-cloner/           clone-contract (the 5 clone checks SC1..SC5, the registration
                           rule — suffix before -vN, no '#' — and what to copy vs share)
  dr-generator/          isaaclab-dr-reference (3 groups: robot/object/obs-noise; full randomize_*
                           function surface, mode→operation map, discovery recipe, read-back recipes,
                           once-per-episode reset rule),
                         smoke-contract (what S7 verifies + substitutions)

knowledge/experiences/             cross-run heuristic ledgers (numbered, append-only)
  rl-tuning-agent/       tuning-experience.md
  task-generator/        task-experience.md
  reward-tuning-agent/   reward-experience.md
  dr-generator/          dr-experience.md
  task-library/          task-design knowledge indexed by embodiment + family (not by agent);
                         one self-contained <task>-<repo>.md probe-task spec per task (+ README.md):
                           manipulation/*.md
                           locomotion/{humanoid, quadrupedal}/*.md
```

### L6a — Process logs (per-run engineering record)

Each agent's per-run process log lives **inside its own subdir** — `<repo>/harbor/rl-integration-generator/history.md`, `<repo>/harbor/benchmark-generator/history.md`, and the per-task `harbor/create-task/<slug>/{task,reward,dr}-history.md`. One short markdown section per run: tool used / agent / command / one-line result. Append-only. (There is no shared `run-log/` folder.)

### L6b — Receipts (end-of-run user summary)

`<repo>/harbor/dependency-generator/install.md`, `<repo>/harbor/benchmark-generator/history.md`, `<repo>/harbor/benchmark-generator/benchmark.md`, `<repo>/harbor/rl-integration-generator/rl-integration.md` — generated by the matching subagent at the end of a run. English only (constraint #1).

### Workspace layout (one root for everything plugin-generated)

```
<repo>/                                user repo root, untouched except for the carve-outs below
├── .venv/                             uv-managed venv (created by dependency-generator's setup_uv.sh)
├── scripts/
│   ├── _<family>_env.py               kept at root — benchmark-generator's smoke-helper convention
│   ├── run_random.py                  kept at root — L1 smoke entry point
│   └── render_random.py               kept at root — L2 smoke entry point
└── harbor/                          single root for plugin-generated artifacts
    ├── dependency-generator/{setup_uv.sh, probe.json, install_plan.json, install.md}   dependency-generator outputs
    ├── benchmark-generator/{benchmark-spec.json, task_overview.md, .task_list.json, history.md, benchmark.md}   benchmark-generator outputs
    ├── rl-integration-generator/{rl-suite-spec.json, rl-integration.md, history.md}   rl-integration-generator outputs (receipts/metadata + L6a process log; the RL training tree stays at harbor/scripts/rl/ etc.)
    ├── rl_experiments/{sweeps,tunes}/<id>/                       sweep + tune workspaces (canonical root)
    ├── create-task/                                            /harbor:task-create workspace
    │   ├── task-implementation.md                                family guide (benchmark-generator output)
    │   └── <slug>/                                               per-task workspace, one folder per /task-create run
    │       ├── spec.json                                         args + per-phase status (orchestrator)
    │       ├── task-history.md                                   §1..§6 design record: per-section Analysis (why) + Validations (what passed)
    │       ├── test-checklist.md                                 every check that actually ran, rendered from the smokes' verdict files
    │       ├── task-analysis.md                                  design rationale only (Validations stripped) — read by §6 + reward candidates
    │       ├── smokes/                                           rendered smokes + <smoke>.verdict.json each writes as it runs
    │       ├── smoke_s{3,4,6}_frames/                            reset layout · per-predicate states · rollout keyframes
    │       ├── reward-history.md                                 verbose log: §6 (reward-tuning-agent)
    │       ├── dr-history.md                                     verbose log: §7 (dr-generator)
    │       └── handoff-dr-generator.md                           §7 handoff: available+effective DR terms per group, modes, ranges, smoke results
    ├── scripts/rl/<impl>/{train,eval,render,env_wrapper}.py      RL training tree
    ├── configs/rl/{ppo,sac,td3}.yaml                             RL configs (Hydra)
    ├── outputs/<algo>_<task>_<ts>/{checkpoint,metrics.jsonl,
    │   tb/, curves/, render.mp4}                                 training artifacts
    └── utils/data_logger.py                                      DataLogger
```

The `harbor/` directory is a valid Python package — rendered scripts use:

```python
REPO     = Path(__file__).resolve().parents[4]   # actual repo root (for scripts/_<family>_env.py)
HARBOR = Path(__file__).resolve().parents[3]   # <repo>/harbor  (for utils.data_logger, configs/rl/, outputs/)
sys.path.insert(0, str(HARBOR))                # so `from utils.data_logger import DataLogger` resolves
sys.path.insert(0, str(REPO))                    # so `from scripts._<family>_env import ...` resolves
```

Hydra `config_path="../../../configs/rl"` is unchanged (3 ups from `harbor/scripts/rl/<impl>/` lands in `harbor/`, then into `configs/rl/`).

### Hooks

- `hooks/pretool_safety_check.sh` — refuses obviously-destructive Bash patterns (force-delete of `/` or `~`, fork bomb, mkfs, …). Emits `hookSpecificOutput.permissionDecision: "deny"` and exits **0**: the decision belongs in the payload, and exiting 2 makes Claude Code take the stderr path and treat the JSON as raw text instead.
- `hooks/stop_audit_log.sh` — appends a one-line audit entry on Stop / SubagentStop. **Opt-in via `HARBOR_AUDIT_LOG=1`.** Hooks fire in every project the user opens, not just benchmark repos, so a logger that defaults to on records unrelated work in the user's home directory unasked.

There is deliberately no PostToolUse output-truncation hook. PostToolUse cannot replace or
suppress a tool result — the output is already in context by the time it runs, and the event
only supports *adding* context. A hook there can make the transcript longer, never shorter.

### Plugin-level permissions

`settings.json` (at plugin root, sibling to `.claude-plugin/`) ships an allowlist for the python / uv / bash subcommands the plugin's subagents need to run unattended. The pretool-safety hook still blocks the destructive cases — the allowlist only removes the prompt; the safety net stays.

### Historical / archived

- `harness-refactor/` — B0-B2 refactor logs, planning docs, deprecated B2 artifacts (`render_manifest.py`, `run_task.py`, `manifest.schema.json`). Untracked. Don't ship; don't reference from runtime code.

See `README.md` for end-user / contributor instructions.

---
> Source: [supersglzc/harbor-rl](https://github.com/supersglzc/harbor-rl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
