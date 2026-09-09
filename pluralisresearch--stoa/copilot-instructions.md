## stoa

> Guidance for anyone (human or AI agent) working in this repo. stoa runs **RL post-training with

# CLAUDE.md: stoa

Guidance for anyone (human or AI agent) working in this repo. stoa runs **RL post-training with
Apple-silicon Macs as the rollout fleet**: **MLX rollout workers** on the Macs generate rollouts, one
**CUDA slime/Megatron trainer** learns from them, and the two sides meet only at a **Cloudflare R2** bucket
(the PULSE weight-sync protocol), never addressing each other directly. Underneath, it's disaggregated,
off-policy GRPO.

## Layout

- `worker/`: the MLX rollout actor (`dRL_producer.py`), int8 decode, the continuous-batching scheduler
  (`cb_scheduler.py`), and the LFM2-only paged-KV Metal kernels (`native_kernels.py`, `paged_*.py`; the
  quickstart's Qwen path uses contiguous KV, so `DRL_KV_MODE=paged_prefix` is a no-op there).
- `common/`: R2 transport (`r2_*.py`), PULSE weight sync (`pulse_delta.py`, `pulse_sync.py`,
  `pulse_chain.py`), and the rollout codec.
- `trainer/`: slime custom functions: the rollout source (`r2_rollout.py`), the DPPO gate
  (`dppo_gate.py`), the Dr.GRPO reducer, replay buffer, staleness filter, eval.
- `patches/`: minimal source patches applied to THUDM/slime (`apply_slime_patches.py`).
- `envs/psqa_search/`: the PaperSearchQA multi-turn-search environment plus trainer launch scripts.
- `ops/`: node operations: one-command Mac join, retriever setup, fleet provisioning, supervisor.
- `bench/`, `scripts/`: benchmark harnesses and the data-fetch/build tooling.

Start at `README.md` (the runnable GSM8K quickstart) and
`runs/psqa-decoupled/REPRODUCE.md` (the reference write-up of the private 8B PSQA run, not a from-scratch recipe).

## Commits: Conventional Commits (required)

```
<type>(<scope>): <imperative subject>
```

- **types:** `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`.
- **scopes** (where useful): `worker`, `trainer`, `common`, `pulse`, `r2`, `slime`, `envs`, `bench`, `ops`, `scripts`.
- Subject: imperative, concise (≤ ~72 chars), no trailing period. Body explains the *why*.
- **Never** add AI/Claude attribution: no `Co-Authored-By`, no "Generated with", no emoji trailer.

## Environment & tooling

- Python **3.12**; **uv** is the package manager. `uv sync --extra worker` (or `retriever` / `data` /
  `dev`) installs a role's dependencies from `pyproject.toml`; `uv.lock` pins exact versions. The
  `retriever` extra (pyserini) requires Python 3.12, so install it into a 3.12 venv (`ops/setup_retriever.sh`
  creates one).
- The **worker** runs on Apple silicon (MLX); set it up with `ops/setup_mac_worker.sh` (creates the venv,
  applies the router patch, runs the int8 parity self-cert). The **trainer** runs inside the pinned
  slime/Megatron container, so its Python comes from the image, not uv.
- Most runtime environment variables use the `DRL_` prefix (historical; kept as the stable config
  contract); `R2_*`, `PSQA_*`, `HF_*`, and `LFM2_*` are also load-bearing.

## Lint & format

Config lives in `pyproject.toml`; run `pre-commit run --all-files` before committing. `ruff` (rules
E/F/B/UP), `black` and `isort` both at line-length 119 (isort `profile=black`). `E402` is ignored because
modules use a `sys.path` bootstrap before imports; keep the explicit `# noqa: E402` on those lines.

## Testing

- `pytest` at the repo defaults must stay **green on any machine**. Tests that need hardware or heavy deps
  guard themselves so they *skip* (not error) when the dep is absent: `pytest.importorskip("mlx.core")`
  for Metal-kernel tests, `importorskip("torch")` for trainer tests.
- The default `pytest` lane is offline and CPU-only. The heavier lanes are opt-in flags a bare run never
  triggers: `--run-realr2` (live Cloudflare R2, needs creds), `--run-metal` (Apple GPU), `--run-slow`
  (long batteries), `--run-realtok` (downloads the HF tokenizer). Never add a test that mutates live
  infrastructure without gating it behind the `realr2` marker (and its `--run-realr2` flag).
- Write **targeted, discriminating** tests: one behavior per test, named for what it proves, asserting the
  outcome that distinguishes the bug from the fix. A test that passes under both the bug and the fix is not
  a test.

## Load-bearing contracts (do not break these)

These are the invariants the system's correctness depends on. Read the module docstrings before touching
the code.

- **MLX lazy-evaluation ordering (`worker/paged_pool.py`, `native_kernels.py`).** The paged-KV pool is
  written in place, so ordering is enforced by hand: a returned **ordering token** is the read-after-write
  edge, **read stamps** guard write-after-read, and freed blocks are parked until a **drain** (`mx.eval`)
  confirms outstanding reads finished. Never wrap the kernels in `mx.compile`, never drop or reuse an
  ordering token, and derive stamps as copies (`+ 0`) so they do not pin parent buffers.
- **One weight version per cohort (`worker/dRL_producer.py`).** The worker pulls and swaps weights only
  *between* generation cohorts, then latches one immutable `(model, tok, version)` snapshot and generates
  the whole batch against it. A rollout group never spans two weight versions.
- **PULSE integrity (`common/pulse_*.py`).** Weights sync as periodic full **anchors** plus
  **sparse-lossless deltas**; integrity is an **xxh3-128** digest chain. A fresh worker cold-joins by
  pulling the newest anchor and composing adjacent deltas to `HEAD`.
- **Off-policy correction (`trainer/`).** Stale rollouts are made trainable by bounded **staleness** and
  **reuse** gates, the worker's recorded behavior logprobs fed through **PPO dual-clip**, and the **DPPO
  gate** (`dppo_gate.py`), the original advantage-conditioned Eq. 12 mask (it blocks only moves *away* from
  the behavior policy in the gradient's direction; corrective moves are never masked). The gate is binary
  (drop a token), never a multiplicative weight. The shipped launcher runs `--get-mismatch-metrics` with the
  gate on the ratio and does *not* enable a separate multiplicative TIS term (`--use-tis` is off).
- **Precision planes.** Rollouts are generated with an int8 worker (group size 64, `int8g64`); the trainer
  computes in bf16. The recorded behavior logprobs and the ratio-based objective (PPO dual-clip + the DPPO
  gate) reconcile the MLX-to-CUDA behavior gap across that boundary.
- **Multi-turn token alignment (`envs/psqa_search`).** Multi-turn episodes are assembled at the
  **token-id** level, not by re-tokenizing text, so the worker's recorded logprobs stay self-consistent
  with what it sampled.

## Module conventions (from the worker refactor)

- **One-way imports.** Dependencies flow in one direction (for example `producer_decode → sweep_telemetry`,
  `paged_cache → paged_pool → native_kernels`); do not introduce a back-edge.
- **Re-export by object**, and integrate with frameworks by **attribute monkeypatch** (see
  `paged_cache.install_paged_lfm2` and `patches/apply_slime_patches.py`) rather than editing vendored
  source in place.

## Comments & documentation

Every module opens with a short "what this is" docstring; public functions and classes are documented;
comments state constraints the code cannot show (the contracts above). **Do not narrate project history in
code**: no dates, run names, or internal codenames. Document the code as it is, for a reader who has never
seen it.

## Type hints

Annotate the public functions and dataclasses in code stoa owns (`worker/`, `common/`, `trainer/`,
`envs/`); they document the R2, rollout, and weight-sync contracts at the call boundary. Two deliberate
exceptions: vendored or adapted files (the Apache-2.0 upstreams) keep their upstream style rather than
being retrofitted, and tests may stay lighter. There is no `mypy` gate (slime and mlx-lm do not type-check
either), so hints are for the reader; `ruff` and `black` enforce the rest.

## Secrets

Never commit credentials (R2 keys, W&B/HF tokens, `.r2env`, `.env`, `.netrc`, `*.pem`). Read them from the
environment at runtime; ship `.example` templates only. They are gitignored.

## Licensing

The framework is **MIT** (`LICENSE`). A few files are vendored or adapted from Apache-2.0 upstreams
(Search-R1, verl, slime); they retain their upstream headers and are listed in `THIRD_PARTY_NOTICES.md`
and `NOTICE`, so keep those notices intact. The reference run's base **model** (LFM2.5-8B-A1B, LFM Open
License) and **datasets** carry their own licenses, which apply to that run, not to the framework; the
trained checkpoint is private and not distributed here, so those redistribution terms are informational.
See the README's License section.

---
> Source: [PluralisResearch/stoa](https://github.com/PluralisResearch/stoa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
