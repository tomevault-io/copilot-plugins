## llm-bootstrap-control-framework

> `policy_bias_lab/` is a DIAGNOSTIC of whether an LLM can author useful task knowledge itself. The

# Agent instructions for llm-framework

## The task-agnosticism rule (hard constraint)

`policy_bias_lab/` is a DIAGNOSTIC of whether an LLM can author useful task knowledge itself. The
framework (orchestrator, arbiter, diagnostics, prompt templates) must therefore contain **no
task-specific information, interpretation, or instructions**. Every piece of task knowledge in the
loop must come from exactly one of two sources:

1. **Injected environment/robot/task data** — the robot spec (`robot_spec(env)`: actuators, ranges,
   groups, world-sign conventions), the environment's own eval fields and their names, and the task
   definitions in `policy_bias_lab/tasks.py` (NL description, success predicate, graded objective,
   progress-metric list). These are *data handed to the framework*, keyed by task, never baked into
   framework logic or prompt prose.
2. **LLM-authored artifacts produced during the run** — context calls (execution account, failure
   modes, kinematics), candidate programs (stages, gates, channels, per-stage `success`
   expressions), and diagnostic `probes`.

Concretely, when working on this code:

- **Never add task nouns or task heuristics to prompt templates** (`policy_bias_lab/prompts/*.md`).
  Templates hold structure and generic mechanism docs only; task content arrives via `$task`,
  `$spec_block`, `$context_block` substitution at runtime.
- **Never add framework-authored interpretation of behavior.** Diagnostics report *measurements*
  (rates, signal means/trends, gate values, hand-off statistics, intent-vs-executed and
  command-vs-measured gaps, probe results) under the environment's own field names. Labeling a
  pattern ("air closure", "flinging", "knocked the object") is the LLM's job in the revision loop.
  If a run exposes a new failure mode, the fix is to surface *more generic evidence* (a new generic
  measurement or a wider probe vocabulary), not to hardcode the diagnosis.
- **Do not carry lessons across runs through the framework.** A discovery made in one run (e.g. a
  useful gate structure, a wrist posture, an exploit) must not be transplanted into prompts,
  defaults, or scoring logic so later runs inherit it. The system has to be able to re-derive it
  organically. Cross-run knowledge lives only in `tasks.py` (task definitions) or the env.
- **New diagnostics must be structural.** They may reference stages, gates, signals, actuators,
  transitions — the model's own authored objects and the injected vocabulary — never what those
  objects are "supposed" to do for a particular task.
- **The signal vocabulary is not framework property.** The framework exposes only RAW observables,
  mechanically enumerated from the env/robot (`raw_signal_fn`: world positions/velocities in obs,
  commanded actuator targets, measured joint positions). Every DERIVED quantity — distances,
  proximity/grip gates, closure fractions, alignment/orientation measures, and their thresholds —
  is task structure and must be LLM-authored per candidate (`signals: {name: expr}`). Do not add
  derived signals to the framework, advertise them in prompts, or tune their thresholds in code:
  that both leaks task assumptions and silently restricts what candidates can express or measure
  (the fixed 8-signal vocabulary made palm orientation literally unrepresentable). The legacy
  derived names (`near`, `gripped`, `closure`, `lift`, `palm_obj_dist`) survive only for
  archived-program replay and must not appear in prompt templates. Probe-only episode-relative
  extras (`obj_disp_xy`, `obj_speed`) are the narrow exception: raw rigid-body quantities that
  need a per-episode baseline the stateless obs cannot carry.

- **Body-motion diagnostics are structural, not interpretive.** `stage_occupancy` reports measured
  kinematics of the observed bodies (speeds, net displacement from the episode-start pose), overall
  and per dominant stage, under the env's own field names. This is mechanical obs-layout data —
  keep it that way: report the numbers, never classify a displacement as "knocking", "disturbance",
  or "progress" in framework code or templates. Diagnosing aggressive-approach/object-pushing
  behavior from these numbers is the LLM's job in the revision loop.
- **Channel SELF observables are structural.** `ctrl_self`/`q_self` in a channel expression bind,
  per actuator in the channel's set, to that actuator's own commanded target / measured joint
  position -- pure obs-layout indexing (`_compile_channel`). Prompt docs may describe what the
  mechanism does (a joint stops tracking its target when blocked; exprs over the gap react to
  that), but must never say which task situations call for it.
- **The embodied procedure context call (`context_procedure.md`) is a mechanism, not content.**
  It asks for an exhaustive physical account of performing `$task` -- including the "obvious"
  unsaid requirements -- and the LLM authors all of it at runtime. Keep the template free of task
  nouns and behavioral advice; its whole point is that gentleness/stillness/contact-order
  knowledge gets SAID by the model, not baked into the framework.
- **Authored EVALS carry only TIE-BREAK selection weight.** A revision may be adopted on an
  authored-eval improvement only when its objective is within the measured noise band
  (`_eval_battery_delta` + the `eval_accept` guard); evals must never override a real objective
  regression -- the model authors its own tests, so unguarded eval-selection is self-grading.
  Do not loosen this guard.
- **The ONE sanctioned prose exception** (user-approved 2026-07-04; extended with user approval
  2026-07-05 and 2026-07-08): the dexterity-conditional gentleness principle in
  `prompts/framework_freeform_staged.md` AND mirrored in `prompts/revise_candidate.md` -- *if the
  model judges the task to be dexterous manipulation*: first contact at near-zero relative speed,
  a ceiling on contact force, only the minimum force each interaction needs, no displacing items,
  hard speed limits for translating body motion, non-oscillating positioning stages, budgets
  expressed as explicit signal/gate/channel conditions. The applicability judgment stays with the
  model. Do not extend it further, add task nouns to it, or add other behavioral advice of this
  kind without explicit user approval. (The applied-force law itself -- servo_gain * (ctrl - q) --
  is injected robot data in `robot_spec`, not prose.)

Known legacy exceptions (quarantined, not in the default path): `policy_bias_lab/legacy/*` and the
open-loop prefilter `prior_eval.score_program` (its `fling_fraction` etc. predate this rule; it is
prefilter-grade only — do not route its labeled fields into prompts).

## Practical notes

- Python: use `.venv/bin/python`. Env init takes ~4 min (JAX/XLA); run env-touching scripts in the
  background. One GPU (8 GB) — run PPO jobs sequentially.
- `experiment_runtime/environment.py` is the repository-owned environment boundary; env-side
  reward/eval changes still require explicit approval.
- Long PPO runs: `policy_bias_lab.cli.long_ppo`; selection runs: `policy_bias_lab.cli.agentic_selection`
  (checkpointed, resumable). Both handle SIGINT/SIGTERM by finishing the in-flight step and saving.

---
> Source: [robin10125/llm-bootstrap-control-framework](https://github.com/robin10125/llm-bootstrap-control-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
