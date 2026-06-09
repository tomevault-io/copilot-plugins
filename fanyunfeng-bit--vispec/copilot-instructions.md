## vispec

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ViSpec — vision-aware speculative decoding for VLMs (LLaVA-1.5/1.6, Qwen2.5-VL). A 1-layer "draft" transformer is trained to predict the target VLM's next tokens; at inference a tree of draft tokens is verified by the target VLM in a single forward pass. The novel piece versus EAGLE/Medusa is the `ImgAdaptor` (`vispec/model/cnets_ours.py:603`), which compresses image tokens into `num_q` query vectors fed into the draft model's attention, plus a global image feature broadcast to text tokens.

Requirements: `python>=3.10`, `transformers==4.51.3` (the `modeling_*_kv.py` files are pinned forks of HF model code with KV-cache hooks — upgrading transformers will break them). `pip install -r requirements.txt`.

## Three-stage workflow

The whole pipeline assumes you produce data, train in two stages, then evaluate. Steps depend on each other; do not skip ahead.

### 1. Data generation (`vispec/ge_data/`)

`allocation_*.py` are dispatchers that split `[--start, --end]` across visible GPUs and spawn `ge_data_all_*.py` workers via `ThreadPoolExecutor`. Workers write per-sample tensors to `--outdir`.

- `allocation_{llava,qwen}_shargpt.py` → text-only data (ShareGPT) for **Stage 2.1**.
- `allocation_{llava,qwen}_pretrain_gen.py` → multimodal data (LLaVA pretrain blip_laion_cc_sbu_558k, with target VLM generating long answers via "Please answer with at least 1000 words.") for **Stage 2.2**.

### 2. Training (`vispec/train/`)

Stage 2.1 — `main.py`: bootstrap draft on text-only data.
Stage 2.2 — `main_mtp.py`: continue with ViSpec on multimodal data; loads stage-1 weights via `--loadpath …/state_20/model.safetensors`. Pass `--use-ours=True --num-q=2 --mtp-steps=1`.
`main_medusa.py` is the Medusa-style baseline.

Both launched with `accelerate launch --multi_gpu -m --mixed_precision=bf16 vispec.train.<main>`. `--configpath` points to a JSON with `num_hidden_layers: 1` and the base model's hidden dims (see `vispec/train/{llava_1.6_7B,llava_1.6_13B,qwen2.5_vl_3B,qwen2.5_vl_7B}_config.json`; `pangu_mm_pi_7B_config.json` is exploratory and not in the documented base-model set); `--basepath` is the HF id of the target VLM. Checkpoints land in `--cpdir/state_<epoch>/`.

`--num-q` MUST match between stage 2.2 training and inference — it determines `ImgAdaptor`'s query count and is baked into the saved weights.

### 3. Evaluation (`vispec/evaluation/`)

Per-benchmark pairs: `gen_baseline_answer_<bench>.py` (target-only autoregressive) and `gen_spec_answer_<bench>.py` (speculative). Benchmarks: sqa, coco_caption, gqa, mme, mmvet, seed_bench, textvqa, vizwiz, vqav2, msvd_qa, mvbench, synthdog, hr_bench, vicuna, mmbench. Per-bench prompt builders live in `*_prompt.py`.

Each spec script switches between three drafts via flags (see `gen_spec_answer_sqa.py:143`):
- `--use-ours=True` → `spec_model_ours.SpecModel` (ViSpec)
- `--use-medusa=True` → `spec_model_medusa.SpecModel`
- neither → `spec_model.SpecModel` (EAGLE-style baseline)

Driver shells run a fixed bench suite end-to-end:
- `baseline.sh --spec_dir <ckpt> --base_model <hf-id> --result_name <tag>`
- `exp.sh` (ViSpec; defaults `depth=3 top_k=8 total_token=30 num_q=2`)
- `exp_eagle.sh`, `exp_medusa.sh`

`phase1.sh` at the repo root is a personal scratch driver (one-off coco_caption triple comparison), not part of the documented workflow — don't treat it as a stable entry point.

Outputs land in `vispec_data/results/<bench>_test/<result_name>/test-temperature-<T>.jsonl`. `--base_model` / `--spec_dir` should be **local filesystem paths**, not HuggingFace repo ids — `SpecModel.from_pretrained` silently falls back to a default config when it can't resolve a path, masking misconfigured runs.

After running, `python -m vispec.evaluation.speed` (edit `baseline_dir`/`result_dir` at top) computes per-method speedup ratios against baseline. `vispec/evaluation/speed_method.py` is a single-file standalone variant for sqa_test sanity checks (hard-coded paths at the top).

`--depth` / `--top-k` / `--total-token` parameterize the draft tree (`vispec/model/utils*.py`, `choices.py`); `--temperature 0` is greedy and `1.0` is stochastic — both are reported in the paper's table.

## Ongoing: PilotSpec follow-on (see `PILOTSPEC_DESIGN.md`, `STEP_C_RUNBOOK.md`, `EXPERIMENTS_LOG.md`, `ANALYSIS.md`)

A redesign that replaces the `ImgAdaptor` compression with explicit image-token selection (sink-detection + middle-layer text→image attention) plus two MLP gates (step-level skip, tree-node early-exit). Four docs together cover it:

- `PILOTSPEC_DESIGN.md` is the **design source of truth** — update it (and bump §12's version row) when the plan changes. §11 maps each PilotSpec piece back to the ViSpec file/line it derives from. Before extending PilotSpec, read §12 (versions) and §9 (risk table) — they encode negative results that supersede earlier sections.
- `STEP_C_RUNBOOK.md` is the **executable runbook** for full Qwen2.5-VL-7B training + SQA eval (Phases 0–6). Follow it command-by-command.
- `EXPERIMENTS_LOG.md` is the **append-only experiment journal** — every training/eval run gets a date + config + result + analysis entry.
- `ANALYSIS.md` is the **append-only root-cause analysis** — paired with the log: when an experiment shows a gap between design assumption and result, add an A-entry here ("假设 → 现象 → 原因 → 待验证"). Index at the top lists which experiments each analysis depends on.

Implementation has progressed past the zero-training harnesses:

- **Draft model** (`vispec/model/cnets_pilot.py`): drops `ImgAdaptor`, accepts pre-selected `image_tokens_K` + `global_img_summary` kwargs, has `mlp_a` (step skip) and `mlp_b` (tree-node score). Llama submodules are imported from `cnets_ours` to avoid divergent forks.
- **Inference entry** (`vispec/model/spec_model_pilot.py`): default single-pass prefill (v9.3) — target prefill captures middle-layer attention via the fork's `capture_attn_layers` mechanism, then `_pilot_select_from_captured_attn` builds `final_idx` from those captured signals (no extra base_model forward). CLI `--two-pass-prefill` falls back to the legacy 2-pass path (`_pilot_select_image_tokens` runs an extra forward). Caches `image_tokens_K` / `global_img_summary` / `image_positions_K` on `spec_layer` for the draft to read. MLP_A `tau_accept_min` gates step skips (`p_accept < tau_accept_min → skip`; default 0.0 = never skip). Tree expansion default is best-first (`cnets_pilot.Model.best_first_genrate`) using MLP_B as the node scorer; CLI `--use-eagle-tree` falls back to EAGLE-style fixed-depth.
- **Training** (`vispec/train/main_pilot.py`): stage 2.1 (text) + stage 2.2 (multimodal w/ MLP_A/B losses). `main_pilot_streaming.py` is a stage-2.1 variant that runs target VLM inline instead of reading a ~1 TB .ckpt cache — preferred when training on the full 68K ShareGPT. PilotSpec training entry differs from ViSpec: `--stage 2.1/2.2` (no `--use-ours / --num-q`), `--mtp-steps 0` (no MTP loop in stage 2.1, contra ViSpec's stage-2.2 trick).
- **Data** (`vispec/ge_data/allocation_qwen_pilot.py` → `ge_data_all_qwen_pilot.py`): stage-2.2 cache (11 fields: ViSpec 4 + PilotSpec 7 incl. `vision_tower_norm`, 3 attention variants, `target_logit_argmax/entropy`, `image_grid_thw`). `simulate_gate_labels.py` regenerates MLP_A/B labels at refresh epochs (subprocess invoked from main_pilot) — atomic rename + manifest JSON for rollback safety. Step C uses depth-1 simplified labels (top-k columns); Step D will switch to multi-depth path labels.
- **Calibration** (`vispec/calibration/profile_vision_norm.py`): hook on `model.visual.blocks[-1]` to profile pre-merger L2 norm and pick `tau_sink`. **Always run before training a new base-model family** — `tau_sink` is model-family-dependent: CLIP ViT-L (LLaVA) ≈ 100 matches the original design default; Qwen2.5-VL pre-merger ≈ **20751.52** (E0 in `EXPERIMENTS_LOG.md`; ~200× larger than CLIP, design default 100 is unusable). Write the p99 into the model's `*_pilot_config.json::tau_sink`.
- **Eval** (`vispec/evaluation/gen_pilot_answer_sqa.py`, `exp_pilot.sh`): only SQA is wired up so far; other benches are mechanical copies waiting to be brought online.

The original zero-training Step A / Step B harnesses (`gen_step_a_answer_sqa.py`, `gen_step_b_answer_sqa.py`, `step_a_hook.py`, `compare_step_a.py`) are kept as regression floors. Both ride on a stock ViSpec checkpoint via `pre_prefill_hook` (Step A) and `spec_layer.tree_depth_tau` (Step B); negative results documented in `vispec_data/results/sqa_test/STEP_B_REPORT.md` and design-doc risk #9.

## Code map

- `vispec/model/spec_model{,_ours,_medusa}.py` — entry classes. `SpecModel.from_pretrained` dispatches on the base model's `architectures[0]` (`LlamaForCausalLM`, `Qwen2ForCausalLM`, `MixtralForCausalLM`, `LlavaForConditionalGeneration`, `LlavaNextForConditionalGeneration`, `Qwen2_5_VLForConditionalGeneration`) to the right `modeling_*_kv` wrapper. `specgenerate` is the speculative loop. `spec_model_ours.py` additionally exposes a `pre_prefill_hook` attribute (None by default); when set it fires inside the prefill path with `(model, input_ids, inputs_embeds, special_image_mask, kwargs)` and must return possibly-modified `inputs_embeds` — used by the Step A harness, not a hot-path extension point.
- `vispec/model/modeling_*_kv.py` — forked HF modeling files, hooked so the draft can read/write the target's KV cache. Edit with care; they shadow the upstream classes.
- `vispec/model/cnets_ours.py` — the ViSpec draft: `LlamaDecoderLayer` (1 layer), `ImgAdaptor` (multi-query attention pooling image tokens to `num_q` slots), and `Model` that ties them together with the target's embedding and lm_head. The `tree_depth_tau` attribute on `Model` (read inside `topK_genrate`) is an injected per-depth cum-logp early-exit cutoff used by the Step B harness; unset → no early exit.
- `vispec/model/utils.py`, `utils_c.py`, `choices.py` — tree buffers, retrieval indices, sampling, and the hard-coded Medusa/EAGLE tree topologies.
- `vispec/model/cnets_pilot.py`, `spec_model_pilot.py`, `utils_pilot.py` — PilotSpec draft (no `ImgAdaptor`, explicit K-token selection, MLP_A/MLP_B gates). Imports Llama submodules from `cnets_ours` to stay in sync. `cnets_pilot.Model.best_first_genrate` (PILOTSPEC §3.4.3 default) and `topK_genrate` (EAGLE fallback, dispatched when `use_best_first=False`) both produce the same EAGLE-compatible buffer shapes so `utils.initialize_tree` / `utils.update_inference_inputs` work without modification. Pilot configs live in `vispec/train/*_pilot_config.json` and carry extra fields (`K_default`, `tau_sink`, `tau_accept_min`, `tau_depth`, `use_best_first`, `attn_train_source`).

## Conventions worth knowing

- Adding a benchmark = add `<name>_prompt.py` + `gen_baseline_answer_<name>.py` + `gen_spec_answer_<name>.py`, then add the bench to the four shell scripts.
- Adding a base model family = add `modeling_<arch>_kv.py`, register it in the `Type ==` chain in `spec_model*.py:from_pretrained`, and add a `*_config.json` with that model's hidden/head dims and `num_hidden_layers: 1`.
- `try: from torch_npu.contrib import transfer_to_npu` blocks exist in training scripts — keep them; they are no-ops on CUDA but required for Ascend NPU runs.
- `torch.cuda.synchronize()` calls around timed regions in eval scripts are load-bearing for the reported speedup numbers; don't remove.
- `--basepath` / `--base_model` / `--spec_dir` / `--model` / data-dir args MUST be local filesystem paths under `/root/autodl-tmp/MODELS/` (and datasets under `/root/autodl-tmp/DATASETS/`) — never HF repo ids. `SpecModel.from_pretrained` silently falls back to a default config when a path doesn't resolve, masking misconfigured runs. Applies to data-gen, training, and eval alike. **`README.md`'s command snippets show HF repo ids for readability — ignore that style and always pass local paths.**

---
> Source: [fanyunfeng-bit/ViSpec](https://github.com/fanyunfeng-bit/ViSpec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
