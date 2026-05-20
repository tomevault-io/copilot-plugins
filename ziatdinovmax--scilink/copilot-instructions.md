## scilink

> Forward-looking design decisions and conventions. Intended for AI assistants

# SciLink — Architecture Notes

Forward-looking design decisions and conventions. Intended for AI assistants
and contributors working on the orchestrator stack. Codebase tour and
per-module docs are elsewhere; this file is about *direction*.

## The mode universe is fixed at three

Every chat-driven orchestrator in SciLink falls into one of three modes —
this is a settled architectural commitment, not a refactoring waypoint:

| Mode | Class | Domain |
|---|---|---|
| `analyze` | `AnalysisOrchestratorAgent` | Experimental data analysis (microscopy, spectroscopy, …) |
| `plan` | `PlanningOrchestratorAgent` | Experimental campaign design |
| `simulate` | `SimulationOrchestratorAgent` | Computational simulations (DFT today, LAMMPS later) |

Anything in scientific workflow falls under one of these three. There will
**not** be a fourth mode. Future capability growth happens *inside* one of
the three, or as a meta-agent on top (see below).

## Capability expansion through skills, not new agents

Going forward, SciLink intends to extend its agentic capabilities primarily through skill
bundles rather than by adding more specialized subagents. New domains,
techniques, or methods are integrated as skill bundles (knowledge +
tools, co-located) under an existing subagent whose shape already fits;
a new subagent class is justified only when its execution structure
itself cannot be expressed within an existing agent. This applies
across all three modes. For example, adding an XRD or Raman skill for 
existing CurveFittingAgent is strongly preferred over creating two new agents
for Ramand and XRD.

## Plan-mode capability boundaries

Two settled conventions on where capability lives in plan mode:

**Plan-mode skills are knowledge-only.** Skill bundles under
`scilink/skills/planning/<name>/` are markdown — no per-skill
`.py` / `TOOL_SPEC` tools. Plan mode reasons and synthesizes; it does
not execute domain numerics. `PlanningAgent` produces plan text, heavy
compute is `BOAgent`'s, and executable artifacts flow through
`generate_implementation_code` — codegen *guided by* the skill's
`implementation` section, so the skill shapes the code rather than
shipping it. Planning subagents deliberately do not consume the
`_shared/_registry` tool inventory. A planning skill that seems to need
a vetted tool is mis-scoped.

**The scalarizer is the lightweight analysis tier.** `ScalarizerAgent`
does simple LLM-generated extraction (pandas / numpy / scipy) over
tabular or otherwise simple data, reduced to scalars plus the BO
input/target schema. It gets no vetted `.py` tools — needing one is the
tripwire that the task is not lightweight and belongs in analyze mode.
Heavy "data → number" extraction is reused, not rebuilt: `run_analysis`
does the hard work with its skill tools, then the scalarizer reduces the
result (`run_analysis → scalarize`). That cross-mode chain is gated on
the future `run_task` contract; until it exists, run the analysis
standalone and feed the resulting scalar in as a data file.

## Why no `BaseChatOrchestrator` refactor

The three orchestrators share a near-identical chat-loop / message-history /
MCP / autonomy / checkpoint shape (~600 lines each). Reflexively extracting
a base class is tempting and **not what we want at this stage**. The rule
of three says abstract on the third copy when the duplication actually
hurts; bug-fix propagation across three files is acceptable cost.

The trigger to do the refactor is "fixes are diverging across copies" or
"a fourth case appears" — neither holds. The fourth case won't appear
(the universe is fixed at three), so the only legitimate trigger is
maintenance pain. We have not hit it.

When building `SimulationOrchestratorAgent`, copy the structure of
`AnalysisOrchestratorAgent`. Don't refactor the other two.

## What the simulate orchestrator looks like

Structure-centric, iterative, two-surface. **Different from analyze mode**
in three ways: no data file required to start, structure-centric
(not analysis-driven), and includes a post-run feedback loop.

### Tool surface

```
Structure phase
  generate_structure(description)             # one cycle, no validator loop
  validate_structure(path)                    # standalone, post-edit re-run
  refine_structure(path, feedback)            # one refinement cycle
  view_structure(path)                        # 3-axis renders

Inputs phase
  generate_vasp_inputs(poscar, request, method='llm'|'atomate2')
  validate_incar(incar, request)              # literature validation
  apply_incar_improvements(...)

Post-run (currently orphaned in the codebase)
  analyze_vasp_output(output_dir)             # OUTCAR / vasprun.xml summary
  suggest_incar_fixes(log_path)               # wraps existing VaspUpdater

Pipeline shortcut
  run_complete_dft_workflow(description)      # what analyze mode exposes today

Session
  list_generated_structures()
  compare_structures(path_a, path_b)
  set_default_calc_params(...)
```

### Session layout

Structure-centric, not analysis-centric:

```
simulate_session_YYYYMMDD_HHMMSS/
├── structures/
│   └── <structure_slug>/
│       ├── POSCAR / INCAR / KPOINTS
│       ├── script_*.py
│       ├── POSCAR_view_{x,y,z}.png
│       └── outputs/        # user drops VASP run results here
├── chat_history.json
├── checkpoint.json
└── session_log.txt
```

### Two surfaces, one agent

Each orchestrator exposes both an interactive and a non-interactive entry
point sharing the same state and tool registry:

- `chat(user_input: str) -> str` — interactive (CLI / UI).
- `run_task(task, context=None, autonomy=None) -> dict` — programmatic
  entry point. Runs one `chat` turn under the requested autonomy mode, then
  derives a structured summary from the session-state delta. `autonomy=None`
  defaults to AUTONOMOUS — the safe choice for a headless caller (never
  pauses for a nonexistent user). A caller attached to a human passes a
  co-pilot / autopilot mode so the sub-agents' human-feedback prompts reach
  that human.

`run_task` is implemented on **all three** orchestrators with that uniform
signature. The return dict shares `status, task, summary, files_produced,
key_findings, suggested_followups, warnings` (plus `error` on failure); the
domain-specific field differs per mode — `analyses` (analyze),
`campaign_state` (plan), `structures` (simulate). This is the contract the
meta agent delegates through.

## The meta agent

The meta agent sits on top of the mode orchestrators so users don't switch
manually — bare `scilink` (or `scilink explore`) launches it. It is **not a
fourth mode**; it's an orchestrator-of-orchestrators with a different role
(router + context bridge). It lives in `scilink/agents/meta_agent/`
(`MetaOrchestratorAgent` + `MetaOrchestratorTools`), copying the
`AnalysisOrchestratorAgent` chat-loop shape.

**v1 scope: analysis + planning.** Simulation delegation is deferred —
`scilink.agents.sim_agents` hard-imports `ase` (an optional dependency), so
the meta module must stay importable without it. `delegate_to_simulation`
is a documented lazy seam: when added, its body does a guarded import
*inside the function*, never at module scope.

### Pattern: agent-as-tool

```
Meta tool registry
  delegate_to_analysis(task, context)   → AnalysisOrchestratorAgent.run_task
  delegate_to_planning(task, context)   → PlanningOrchestratorAgent.run_task
  summarize_session_state()             → cross-specialist status
  get_delegation_history(limit)         → the delegation ledger
  delegate_to_simulation(...)           → deferred lazy seam (not built)
```

There is **no `bridge_context` tool**. `run_task` already accepts a
`context` dict; the meta LLM bridges modes by reading a prior result via
`get_delegation_history` and threading its `key_findings` / `files_produced`
into the next delegation's `context`. The delegation ledger is the
supporting structure.

### Two autonomy levels, not three

The individual modes have a three-level autonomy paradigm (co-pilot /
autopilot / autonomous); `MetaMode` has only **AUTOPILOT** (default) and
**AUTONOMOUS**. A delegation runs the child through its one-shot `run_task`
— a single turn. Co-pilot's model is "pause after every step, wait for the
user's next message," which needs many turns, so it cannot complete a
delegated task. AUTOPILOT and AUTONOMOUS each finish a task in one turn:
AUTOPILOT still pauses at the child's decision points (approve / edit plans
and outputs) via `input()`-based human-feedback prompts — which compose with
`run_task` because they block-and-resume *within* the turn — while AUTONOMOUS
runs end to end. The three-level paradigm is untouched for the standalone
`analyze` / `plan` / `simulate` modes.

### Persistent children, nested sessions

The meta keeps **one persistent child per mode** — lazily created on first
delegation, reused across all delegations so context accumulates — in fixed
sub-directories `<meta_session>/analysis/` and `<meta_session>/planning/`.
After a meta restore a child is re-created with `restore_checkpoint=True`
simply by probing for its `checkpoint.json`. **Each delegation runs the
child under the meta's own autonomy mode** — passed as `run_task`'s
`autonomy` arg (mapped by enum name); the child's resting mode is
irrelevant. So an autopilot delegation keeps the specialist's human-feedback
prompts, which surface to the user driving the meta exactly as in a direct
single-mode session. The planning child is built in CO_PILOT with
`data_dir=None` — the one construction mode that does not require
`data_dir`; `set_autonomy_level` does not re-validate it on the per-call
switch. Per-delegation
isolation: each `run_task` writes into its own sub-directory so a reused
child does not overwrite earlier outputs (analysis already stamps result
dirs; the planning orchestrator writes to a per-delegation
`delegations/<NN>_<slug>/`).

Because the meta consumes children through their `run_task` contract (not
through inherited internals), no base class is required. The contract is
duck-typed; what the children share is *interface shape*, not
*implementation*.

## Sequencing — hard features first, UI later

Engineering philosophy on this codebase: implement load-bearing logic
first, surface it in CLI / UI later. Reasons:

- UI shaped against an unbuilt feature gets reshaped
- Backend logic is independently testable; UI work depends on it
- Simulating the user's flow without a backend produces wishful UX

Concretely, when the simulate orchestrator work starts, the order is:

1. `SimulationOrchestratorAgent` (copy of analyze structure) +
   `simulate_orchestrator_tools.py` with the granular DFT tool registry
2. `scilink simulate` CLI flesh-out (replace the "Coming Soon!" stub)
3. HPC backend (`scilink/hpc/` — `Connection`, `Scheduler`) — see PR #140;
   self-contained module, no orchestrator dependency
4. HPC tools on the orchestrator (`submit_vasp_job`, `check_job_status`,
   `download_results`, …) wrapping #3
5. UI — sidebar mode, chat panel, possibly a wizard surface coexisting
   with the chat surface

The meta agent (`scilink/agents/meta_agent/`) was built following this same
backend → CLI → UI order, over the analysis and planning orchestrators;
simulation delegation is wired into its lazy seam once that path is stable.

## Connection between modes today

Analyze mode connects to DFT via two tools in
`analysis_orchestrator_tools.py`:

- `recommend_dft_structures` — generates DFT structure recommendations
  from cached analysis text via `RecommendationAgent`
- `run_dft_workflow` — runs the full `DFTOrchestrator` pipeline; takes a
  `structure_description` (free text) or `recommendation_index` (pulls
  from stored recommendations)

When the simulate orchestrator ships, **these stay**. Analyze mode keeps
the one-shot pipeline tool because that's the right shape for "I'm done
analyzing, prepare a calc". Simulate mode adds *granular* alternatives
for iterative work. Don't replace `run_dft_workflow`; add alongside.

## Skill subsystem

Skills are domain-specific LLM context shared across the experimental and
simulation agents.

- **Skill bundles** at `scilink/skills/<domain>/<name>/` — one folder per
  skill containing `<name>.md` plus optional sibling `.py` helpers
  (Anthropic-Skill shape).
- **Cross-skill helpers** at `scilink/skills/_shared/` — modules referenced
  by multiple bundles, plus the `_registry.py` / `_spec.py` discovery
  infrastructure.
- **Non-skill utilities** at `scilink/utils/`. The legacy `scilink/tools/`
  no longer exists.

Skill markdown begins with an optional `---`-delimited YAML frontmatter
block. The only field consumed today is `description` (rendered into the
orchestrator's `run_analysis` tool parameter blurb). Add fields only when
there's a consumer; don't accumulate metadata speculatively.

Section vocabulary is **fixed**: `overview`, `planning`, `analysis`,
`interpretation`, `validation`, `implementation`. Off-vocabulary `## headings`
are preserved under `extras` and a warning is logged so authors get
feedback instead of silent loss. The fixed set is load-bearing — controllers
inject specific sections at decision points
(`_get_skill_context(section="planning")`), which is how prompts stay tight.

Multi-skill is end-to-end. `analyze(skill=...)` and the `run_analysis` tool
both accept `str | list[str]`. `TOOL_SPEC` declarations inside a skill
bundle are visible to the LLM only when that skill is active; `_shared/`
specs are always-on (filtered by their `agents=` tag).

Code blocks inside skill markdown are **LLM-facing reference**, not
executable surfaces — the loader does not extract or run them. Runnable
code lives in sibling `.py` files and is registered via `TOOL_SPEC`.
Domain scientists who write markdown only can ship a skill as a single
`<name>.md` and never touch Python; the engineer-maintained helpers
co-locate as siblings.

### Comparison with Anthropic Skills

|  | Anthropic | SciLink |
|---|---|---|
| Folder bundle layout | ✓ (`SKILL.md` + siblings) | ✓ (`<name>.md` + siblings) |
| Description-based selection by the model | ✓ (system prompt, every turn) | ✓ (`run_analysis` tool param, when routing) |
| Section vocabulary | Free-form | Fixed six-section; off-vocab content captured under `extras` |
| Injection granularity | Whole `SKILL.md` once activated | Per-decision via `_get_skill_context(section=…)` |
| Bundled scripts | Model can read and run | Reference-only in markdown; runnable code as sibling `.py` registered via `TOOL_SPEC` |
| Multi-skill loading | Implicit (model loads whichever descriptions match) | Explicit (`skill: str \| list[str]`); active set gates tool visibility |
| Shared library across skills | Not a concept (skills are independent units) | `scilink/skills/_shared/` — always-on infrastructure |

Conceptually: Anthropic skills are *independently distributable units the
model picks at conversation time*; SciLink skills are *in-package knowledge
bundles selected by orchestrator tool routing*, with skill-gated tool
visibility doing what skill activation does upstream. The `_shared/`
carve-out is a deliberate adaptation for in-package code reuse — Anthropic
users would either duplicate the helper or split it into a standalone skill.

## Conventions for prompt patches

When live traces surface bad LLM behavior:

- Encode the **principle** in one short sentence, not a list of phrases
  pulled from the trace. Example-driven prompts overfit and accumulate
  dead weight.
- Trace specifics belong in the commit message and PR description, not
  the prompt itself.
- If a single sentence isn't enough, the rule probably needs structural
  support (a schema field, a separate validation pass) rather than
  more prose.

## Branch hygiene

Non-trivial features start on a dedicated branch off `main`
(`git checkout -b <feature-name>`), never on `main` directly. UI / CLI
exposure for a backend feature can land in the same branch as the
backend, or split into a follow-up PR — depends on review surface area.

---
> Source: [ziatdinovmax/SciLink](https://github.com/ziatdinovmax/SciLink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
