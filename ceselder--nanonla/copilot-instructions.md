## nanonla

> - **This is an open-source repo.** Only standard libs: `pathlib.Path`,

# NLA — instructions for Claude / AI assistants

## Constraints

- **This is an open-source repo.** Only standard libs: `pathlib.Path`,
  `pyarrow`, `transformers`, `datasets`, `httpx`, `pyyaml`, `numpy`, `orjson`,
  `safetensors`, the public `anthropic` SDK.
  No private/internal dependencies.
- Use argparse for CLIs in `nla/`.
- Storage and completion-provider backends are pluggable via import-path
  strings (`--storage-cls`, `--provider-cls`). The shipped implementations are
  `LocalStorage` and `AnthropicProvider`. Cloud storage / other LLM APIs are
  bring-your-own — don't hardcode bucket paths or vendor SDKs into `nla/`.

## Key invariants (do not break these)

- **Data-gen NEVER normalizes** — all parquets store raw vectors
  (`norm="none"`). `stage3_build` asserts input `norm == "none"`. Normalization
  happens at injection time (`injection_scale`) and at loss time (`mse_scale`),
  both read from the sidecar.
- **Stage-1 split is DOCUMENT-level** — partition by unique `doc_id`, all rows
  from the same doc go to the same bucket. Never split positions from one doc
  across `av_sft` / `ar_sft` / `rl`.
- **Stage-0 `_MIN_POSITION = 50`** — need enough left-context for the
  activation to be meaningful. Earlier positions decode to noise.
- **Critic extraction is suffix-anchored** — no scan, no marker token. The
  critic prompt template ends with `... <summary>`; training extracts at
  `tokens[-1]`. `critic_suffix_ids` in the sidecar is for sanity-checking only.
- **Per-doc keyed RNG** — same `(seed, doc_id)` → same sampled positions
  regardless of chunk boundaries, slice ordering, or process count. This is
  what makes multi-GPU stage-0 sharding bit-reproducible.
- **Injection hook scans for the token ID inside the hook** (`inputs[0]`), not
  from precomputed positions. Batching/reordering means precomputed indices
  are wrong by construction.
- **Sidecar is the contract.** Token IDs, prompt templates, `injection_scale`,
  `mse_scale`, `d_model` — all loaded from `nla_meta.yaml` and asserted
  against the live tokenizer at startup. Never hardcode them.

## SFT: TRAIN ON THE FULL THING

- **⚠️ DON'T FORGET THIS — warmstart SFT (AV and AR) must train on EXACTLY 1
  epoch of all ~247k rows (or more).** Pass **`--epochs 1`** to
  `nla/train_sft.py` — it computes `steps = ceil(rows / (batch×grad_accum))`
  from the real loaded row count, so it's one exact full pass (≈15.5k steps at
  batch 16; the AV/AR splits differ slightly in row count, hence "compute it,
  don't hardcode"). Do NOT carry over a small `--num-steps` (e.g. 1000 ≈ 6.5%
  of 247k): that trains on a sliver AND floors the cosine LR early, badly
  underselling FVE. If `--num-steps` covers < 1 epoch the trainer prints a loud
  `WARNING: ... only X% of ONE epoch` at startup — heed it. (Learned the hard
  way on Gemma-4: 1000 steps gave AR FVE 48.4% on a fraction of the data;
  retrained on the full epoch for the real number.)

## RL training: two trainers

- The **verified recipe is single-GPU 4-bit** with HF `generate()` rollouts:
  `nla/train_rl_self_contained.py` (see `scripts/sbatch_rl_fixed.sh` for the
  working invocation). This is the only RL trainer behind any reported number.
- `nla/train_rl_vllm.py` is an **EXPERIMENTAL, unverified** faster-rollout path
  (vLLM via vllm-lens). It refuses to run without `--i-understand-experimental`
  and has documented gaps: (1) vllm-lens norm-matches injection against the
  layer's MLP-delta output, not the full residual — magnitude differs from the
  HF hook, not empirically reconciled; (2) its critic loader needs a full merged
  `NLACriticModel` dir, not the AR-LoRA format `train_sft --use-lora` produces;
  (3) `--train-critic` co-trains the full critic backbone, not LoRA-only. Don't
  report numbers from it without a GPU equivalence check vs the self-contained
  trainer first.
- For vLLM rollouts: use `--tensor-parallel-size N` to spread the rollout
  engine across all GPUs. The HF trainable side stays on one GPU (LoRA's
  ~120M trainable params don't benefit from FSDP), so GPUs 1..N-1 are
  vLLM-only during training-time forward but used during rollout.
- Weight broadcast via `llm.collective_rpc("load_weights", ...)` handles
  TP-sharding internally — same API regardless of TP size.
- If we ever move to full fine-tuning instead of LoRA, sharded training
  (FSDP) becomes worthwhile and the weight-gather path needs the TRL
  `_sync_fsdp{1,2}_params_to_vllm()` treatment (`gather_if_zero3`,
  `summon_full_params`, etc.) — note for future-Claude.

## Debugging

If injection silently fails the actor sees the literal CJK marker char and
free-associates Chinese. Grep generated text for CJK — that's the loudest
smoke test for the entire injection path. Usual causes: the marker token ID
drifted (wrong tokenizer/sidecar), the hook never registered (wrong layer
attribute path), or the prompt template lost the marker char.

---
> Source: [ceselder/nanoNLA](https://github.com/ceselder/nanoNLA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
