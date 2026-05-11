## latentskill

> Compress SkillRL's text skills into trainable latent tokens. Use RL to directly optimize skill representations (text skills are non-differentiable; latent tokens are). Target: NeurIPS.

# Latent Skill RL Project

## Goal
Compress SkillRL's text skills into trainable latent tokens. Use RL to directly optimize skill representations (text skills are non-differentiable; latent tokens are). Target: NeurIPS.

## Current Architecture (Plan C - end-to-end Composer)

Two copies of the same base model:
- **Actor** (FSDP-wrapped, 7B frozen params): standard verl RL pipeline
- **Encoder copy** (non-FSDP, frozen, +14GB/GPU): used by Composer for skill encoding

Composer (`src/skill_composer.py`):
- Holds learnable `query_latents` (2 × 3584 = 7K params, the ONLY trainable parameter)
- `encode_skills()`: appends `query_latents` to skill text embeddings, forwards through Encoder copy, extracts last k hidden states
- Pretrained via SFT distillation (Stage 1, already done → `data/debug_composer/trained_query_latents.pt`)

Integration:
- Actor's `embed_tokens.forward` is monkey-patched: SKILL token positions replaced with Composer's latent output (keeps gradient)
- Actor's `lm_head`: NOT patched; v2 checkpoint has `lm_head[SKILL_tokens] = 0` so model naturally won't generate them
- vLLM rollout: `fsdp_vllm.__enter__()` calls Composer, writes latents into vLLM's `VocabParallelEmbedding` with TP-aware indexing
- Actor `update_policy`: re-encodes per micro-batch to keep fresh gradient graph across backward passes

## Key Paths

| Path | What |
|------|------|
| `/work/nvme/bdns/xzhou10/checkpoints/latent_skills_fullparam/global_step_116` | Base SFT checkpoint |
| `/work/nvme/bdns/xzhou10/checkpoints/latent_skills_token_v2` | v2 expanded vocab (88 SKILL tokens, untied LM head, output=0) |
| `data/debug_composer/trained_query_latents.pt` | Stage 1 Composer output |
| `data/debug_composer/latent_skill_library.pt` | Encoded latent library for all 44 skills |
| `SkillRL/` | verl framework (we modify) |
| `src/skill_composer.py` | Composer + monkey-patch setup |
| `src/expand_vocab_with_skills.py` | Checkpoint vocab expansion |
| `src/train_composer.py` | Stage 1 training script |
| `scripts/debug_rl_e2e.sbatch` | Debug RL (5 epochs, 2h) |
| `scripts/expand_vocab_v2.sbatch` | Regenerate v2 checkpoint |

## Running Training

```bash
# Submit debug RL (Plan C)
sbatch scripts/debug_rl_e2e.sbatch

# Check status
squeue -u xzhou10

# Check logs
tail -50 logs/debug_e2e_<JOBID>.err
grep "training/global_step" logs/debug_e2e_<JOBID>.out | tail -10
```

## Infrastructure (DeltaAI)

- Partition: `ghx4` (GH200 120GB GPUs)
- Account: **`bfdz-dtai-gh`** (new; old `bdns-dtai-gh` hit QOSGrpBillingMinutes)
- Storage:
  - `/work/nvme/bdns/xzhou10/`: **FULL** (old project storage, read-only in practice)
  - `/work/nvme/bfdz/xzhou10/`: active (writable, ~200GB free)
  - HF cache + checkpoints new outputs → `bfdz`

## Critical Version Pins

| Package | Version | Why |
|---------|---------|-----|
| vLLM | 0.19.0 | System provides, can't change (aarch64 wheels limited) |
| transformers | 4.57.3 | Needs `ALLOWED_LAYER_TYPES` for vLLM 0.19.0 |
| tokenizers | 0.22.0 | required by transformers 4.57.3 |

## Modified verl Files

Compared to upstream SkillRL/verl:
- `verl/utils/vllm_utils.py` — try/except for LoRA imports (vLLM 0.19 moved modules)
- `verl/workers/sharding_manager/fsdp_vllm.py` — `get_tp_group()` fallback + Composer sync
- `verl/workers/actor/dp_actor.py` — tensordict `.keys()` + per-micro-batch Composer re-encode
- `verl/workers/critic/dp_critic.py` — tensordict `.keys()`
- `verl/trainer/ppo/ray_trainer.py` — tensordict `.keys()`
- `verl/workers/fsdp_workers.py` — Composer initialization + encoder copy loading
- `verl/workers/rollout/vllm_rollout/*.py` — (deleted) bad_words/logit_bias hacks from v1

## Config Pattern (SLURM sbatch)

Pass Composer config via Hydra overrides:
```bash
+actor_rollout_ref.composer.latents_per_skill=2
+actor_rollout_ref.composer.pretrained_query_latents=PATH
+actor_rollout_ref.composer.skills_json_path=PATH
+actor_rollout_ref.composer.skill_token_map_path=PATH
+actor_rollout_ref.composer.lr=5e-3
```

## Memory Budget (4 × GH200 120GB)

| Component | Per GPU |
|-----------|:-:|
| Actor FSDP shard | ~3.5GB |
| Ref FSDP shard (offloaded) | 0GB |
| Encoder copy (Plan C) | 14GB |
| vLLM (50%) | 60GB |
| Activations + workspace | ~30GB |
| **Total** | **~108GB / 120GB** |

If tight: reduce `gpu_memory_utilization` from 0.5 to 0.4.

## Hyperparameters (matching SkillRL)

RL:
- lr = 1e-6 (for actor; ignored in Plan C where we only train query_latents at 5e-3)
- train_batch_size = 16
- val_batch_size = 64 (or 32 if OOM)
- group_size = 8 (or 4 if OOM)
- max_prompt_length = 4096
- max_response_length = 512
- epochs = 150
- KL coef = 0.01
- top_k retrieval = 6

## Known Issues / Watchouts

1. **`/work/nvme/bdns/xzhou10` is FULL** — any write goes to `bfdz`
2. **wandb entity**: `medagent` (personal entity disabled)
3. **Ray OOM monitor**: disabled via `RAY_memory_monitor_refresh_ms=0` to prevent aggressive worker killing
4. **Dataset cache**: verl's `rl_dataset.py` calls `datasets.load_dataset` which does disk check via `shutil.disk_usage`. NFS reports 0 free on bdns → must use bfdz paths.
5. **Meta tensor trap**: `init_context` uses `accelerate.init_empty_weights` on non-rank-0 workers. Any real tensors created inside it become meta. Put Composer init OUTSIDE `with init_context()`.
6. **FSDP lm_head gather**: calling `Qwen2ForCausalLM.forward` on FSDP-wrapped module with `inputs_embeds` sometimes fails to gather lm_head. Plan C solves this by using `model.model` (Qwen2Model, skips lm_head).

## Paper Story

**Problem**: Agent RL with text skills (SkillRL) has three limits:
1. High token cost (~800-1200 tokens per step)
2. Non-differentiable (RL can't optimize skill content)
3. Not modular (skills baked into model weights after RL)

**Contribution**:
1. Differentiable skills: latent tokens are continuous, RL reward directly optimizes `query_latents`
2. Modular: base model frozen, skills are pluggable SKILL tokens
3. Token-efficient: 2 tokens per skill vs 800-1200 tokens

**Compare against**:
- SkillRL (text skills + full-model RL, 89.9% on ALFWorld)
- SkillZero (skill internalization via curriculum, different approach)
- LatentMem / TokMem (similar infrastructure but SFT only, not RL)

## Related Work References

- SkillRL paper: `related work/SkillRL.pdf`
- LatentMem repo: `LatentMem/` (cloned locally for reference)
- TokMem repo: `TokMem/` (cloned locally for reference)

---
> Source: [WillChow66/LatentSkill](https://github.com/WillChow66/LatentSkill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
