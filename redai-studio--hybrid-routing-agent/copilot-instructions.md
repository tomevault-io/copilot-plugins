## hybrid-routing-agent

> Guidance for AI coding agents (and fast-moving humans) working in this repo. Read this before editing; it encodes the architecture, the invariants, and the footguns that are not obvious from the tree.

# Agent guide (AGENTS.md)

Guidance for AI coding agents (and fast-moving humans) working in this repo. Read this before editing; it encodes the architecture, the invariants, and the footguns that are not obvious from the tree.

## What this is

Code companion to the paper *"Screenshots or Tools?"* ([arXiv:2608.03327](https://arxiv.org/abs/2608.03327)): a hybrid GUI+MCP computer-use agent on OSWorld, evaluated on OSWorld-MCP (309 tasks), plus multi-turn GRPO training. `REPRODUCE.md` maps every paper claim to a command — treat it as the spec.

## Architecture (where the real logic lives)

```
scripts/run_{mcp,puregui}_eval.sh      # eval entry points (env-var configured)
osworld_rl.py                          # RL orchestrator: serve → rollout → reward → preproc → train
├─ OSWorld-main/rl_rollout_local_qwen3vl.py   # rollout worker (per-GPU)
│   └─ OSWorld-main/mm_agents/hybrid_agent_local.py   # ★ THE agent: prompt template,
│        unified <tool_call> action space, sliding window, drop rule, tool retrieval
├─ OSWorld-main/desktop_env/           # VM control + MCP client injection
├─ agents/tool_retriever.py            # BM25 top-18 tool retrieval (fallback path)
├─ reward/osworld_rl_reward.py         # returns, groupwise z-score, 1/T broadcast, dense bonus
└─ train/osworld_train.py              # accelerate + DeepSpeed ZeRO-3 GRPO worker
```

`hybrid_agent_local.py` is the file most questions end at — the message array (Appendix C of the paper) is built there.

## Configs: the #1 trap

One experiment = one YAML in `configs/experiments/`, inheriting `configs/osworld_rl.yaml` via a `defaults:` list (plain OmegaConf merge in `train/utils.py:get_config` — NOT Hydra).

**The base config's values do NOT match the paper.** Only the experiment YAMLs (`outcome_only_rl.yaml`, `context_rl.yaml`, `dense_bonus_rl.yaml`) carry the paper hyperparameters (lr 5e-6, β 0.02, max_steps 50, 4 updates/step). Never quote or copy defaults from the base YAML as "the paper setting". Each experiment YAML's header documents its hypothesis and read-out criteria.

## Data: three sets at runtime, nothing else trains

```
train : train_v2_signal_v2.json (74)   — all RL runs (dense-bonus uses its top24 subset)
monitor: held_out_v2_eval.json (48)    — greedy probe every few steps; NEVER trained on
eval  : test_all_no_internet.json (309)— incl. 89 chrome/multi_apps tasks, true OOD, NEVER trained on
```

Everything else under `evaluation_examples/` and `data/splits/` is provenance or measurement metadata (see `data/README.md`). The `ctx_degraded_*.json` subsets are **pre-registered and frozen** — do not regenerate or "fix" them.

## Invariants (do not break silently)

- **Unified action space**: the model emits exactly one `<tool_call>` per step — either `computer_use` (11 primitives) or one retrieved MCP tool. One MCP call per step, results truncated to 1500 chars.
- **Coordinates**: Qwen3-VL emits relative coords on a 1000-grid; the harness resizes (`coordinate_resize.py`, currently hardcoded to 1920×1080). A wrong coordinate route silently tanks accuracy — this was historically the single worst bug class in the project.
- **Drop rule** (`context_policy=skip_on_mcp_success`): replaces the post-tool screenshot with a text placeholder only on execution-level success (`is_error=false`), not semantic success.
- **Checkpoints**: real snapshots are `runs/<proj>/ckpt/epoch-N-policy/` (complete loadable model dirs).
- **On-policy check**: a healthy run's first window shows `clip=0 / ratio=1.0`, `[DS-CHECK] global_steps` increments, `[CKPT-SYNC]` appears.

## Footguns

- `resume_step.txt` survives `start_from_scratch` — archive the old run dir before relaunching, or the run silently resumes.
- Rollout `--max_tokens` is hardcoded to 8192 in `osworld_rl.py` (the base YAML's 2048 is dead).
- `monitoring/` and `sample/` are intentionally not shipped; the code treats both as optional. Don't "fix" the imports by inventing those packages.
- RL rewrites 0.4–1.8 TB of `.pt` per step: always launch with `OSWORLD_LOCAL_TEMP=<fast local disk>`.
- `results/` and `baselines/` are the paper's evidence base — regenerate via the documented pipelines or don't touch; never hand-edit numbers.

## Cheap feedback loops (use these before anything expensive)

- Syntax/wiring: `python -m compileall osworld_rl.py agents train reward prompts scripts`
- End-to-end smoke (needs VM + weights): `TEST_META=evaluation_examples/smoke_test.json bash scripts/run_mcp_eval.sh`
- Evaluate a checkpoint without training: probe recipe = experiment YAML with `total_step=0 + step0_held_out_eval=true + policy_model → ckpt dir` (see `configs/experiments/probe_*.yaml`).
- Verify shipped result numbers: `python3 results/context_rl/recompute.py`.

## Doc map

| Question | Read |
|---|---|
| Does the code match the paper? | `REPRODUCE.md` |
| Which split file? | `data/README.md` (TL;DR table) |
| VM / MCP-injection setup | `docs/ENVIRONMENT.md` |
| Where a headline number comes from | `results/context_rl/PROVENANCE.md`, `results/{dense_bonus,outcome_only}/README.md`, `baselines/COMPARISON.md` |
| What changed vs upstream OSWorld | `OSWorld-main/PATCHES.md` |

---
> Source: [redai-studio/hybrid-routing-agent](https://github.com/redai-studio/hybrid-routing-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
