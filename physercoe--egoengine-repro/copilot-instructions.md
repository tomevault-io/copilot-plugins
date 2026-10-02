## egoengine-repro

> Reimplementation of "EgoEngine: From Egocentric Human Videos to High-Fidelity

# EgoEngine Reimplementation Workspace

Reimplementation of "EgoEngine: From Egocentric Human Videos to High-Fidelity
Dexterous Robot Demonstrations" (arXiv:2606.12604). No official code exists;
everything here is a from-scratch reproduction. Sim-only is the target.

## Hard rules

- **Package manager: `uv` only** (`uv venv`, `uv pip install`). No conda, no
  system pip.
- **Per-stage venvs** named `.venv-<stage>` at the project root
  (e.g. `.venv-recon`, `.venv-action`, `.venv-policy`). Do not share one venv
  across pipeline stages; dependency sets conflict.
- **Target stack**: Python 3.11, torch 2.7.0+cu128
  (index `https://download.pytorch.org/whl/cu128`), Isaac Sim 5.1.0.0,
  IsaacLab 0.54.2 (=2.3.2). Migrate older stacks to this when feasible.
- **Artifacts stay inside the project**: datasets under `data/`, results under
  `outputs/`. Never write to `/tmp` or $HOME for project data.
- **Shared assets are symlinked, never re-downloaded**:
  - `/app/models/{sam2,mano,hamer,...}` → linked under `data/models/`
  - datasets live under `/app/datasets` → link into `data/` when used
  - HF cache is shared at `~/.cache/huggingface`
- **GPU discipline**: one heavy GPU job at a time. Local GPU is 1× RTX 5080
  16GB (sm_120) — check `nvidia-smi` first. A remote GPU server
  (e.g. 4×A800 80GB, ephemeral) may be used ONLY on GPUs verified free at
  run time; sync outputs back to the local machine; launch remote jobs with
  `setsid nohup ... </dev/null &`.
- **Log all work in `docs/PROGRESS.md`** with dated entries; write a dedicated
  `docs/<TOPIC>.md` per stage so others can reproduce/debug it.
- Also read `/app/project/AGENTS.md` (workspace-wide rules and Isaac gotchas).
- Third-party code is vendored as shallow git clones under `third_party/`
  (with `GIT_LFS_SKIP_SMUDGE=1` to avoid bulk LFS pulls).

## Network

- github/pypi go through a local HTTP proxy (`http(s)_proxy` env vars).
- HuggingFace: use `HF_ENDPOINT=https://hf-mirror.com` with **all** proxy vars
  unset (huggingface.co is unreachable; `all_proxy=socks` breaks
  huggingface_hub). No HF auth token available — only download public,
  ungated models.
- Prefer latest stable versions of everything.

## Layout

- `docs/` — PROGRESS.md (work log), COMPONENTS.md (component inventory),
  ENV.md (venv recipes + gotchas), DATA.md (data inventory), SMOKE.md (smoke
  matrix), TACO_LOADER.md (TACO formats + loader), RETARGET.md (mink IK stage),
  requirements-.venv-*.txt (frozen pins)
- `data/` — datasets + symlinked shared models (`data/taco` → /app/datasets/taco)
- `outputs/` — generated demonstrations, trained policies, renders
  (`outputs/retarget/`, `outputs/taco_check/`, `outputs/smoke/`)
- `scripts/` — runnable pipeline entry points (validate_taco_loader.py,
  run_retarget.py, smoke_*.py + _smoke_run.sh runner pattern)
- `src/egoengine/` — our own package (`taco/` loader+MANO+pseudo-base,
  `retarget/` IK)
- `third_party/` — vendored upstream repos (shallow clones)
- `assets/` — Inspire hand URDFs (dex-urdf copy is the one used by retargeting)

## Status (2026-07-27)

TACO data complete (16 triplets); loader + MANO + pseudo-base validated;
mink IK retargeting done for ALL 16 sequences (thumb-limited residuals).
IsaacLab scene + position-target refine pipeline done; replay+MPC scaled to
all 16 sequences full-frame (8/16 success each, `outputs/refine/summary_16.md`,
failure taxonomy + 97 pending_rl chunks in `docs/REFINE.md` §7). RL residual
stage delivered but under-budget (docs/RL.md). Visual branch done
(docs/VISUAL.md). Remaining: A800 RL warm-start for pending_rl chunks,
HPT policy (flow-matching decoder must be implemented — vendored HPT lacks it).

---
> Source: [physercoe/egoengine-repro](https://github.com/physercoe/egoengine-repro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
