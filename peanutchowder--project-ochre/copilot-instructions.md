## project-ochre

> Generates a multi-row PNG per context: decoded target reference / Model AR / Oracle (optional) / token agreement heatmap / edge maps. Companion `metadata.json` has full 20-frame quality curve against the decoded target-token reference.

# AGENTS.md — Project Ochre Workflow

## Workflow

| Step | Who | Action |
|------|-----|--------|
| 1 | **User** | Implement changes locally (model, training loop, eval tools) |
| 2 | **User** | Push to remote and pull onto GPU instance |
| 3 | **User** | Start training run on GPU instance |
| 4 | **User** | Stop training; download `wandb` run logs and `eval/runs/train/` outputs |
| 5 | **Agent** | Read logs; identify best checkpoint step (transition recovery metrics, not just loss) |
| 6 | **User** | Download selected checkpoint `.pt` file to MacBook Air |
| 7 | **Agent** | Run eval suite (see below); interpret results; recommend next experiment |
| 8 | **Agent** | Evaluate resuls in context of evals. Why did an assumption work/not work?

---

## Checkpoint Selection Heuristics (Step 5)

Primary signal: `static_to_jump_post_au` — unique predicted tokens at the final step of a 20-step rollout after a jump action. Higher = better transition recovery. Secondary: `avg_max_prob` (healthy ≈ 0.25–0.40), `consistency_score`.

Do **not** select purely on CE loss — it does not correlate with transition-recovery quality.

---

## Context Ramp-Up

Before making recommendations, review the relevant project history and artifacts:

1. Read `./changelogs/temporal_changelog.md` for the long-horizon evolution of hypotheses and outcomes.
2. Read the latest relevant files in `./detailed-changelogs/` for the version under review, then the immediately preceding 2–4 versions for local context.
3. Review the matching train-time diagnostics in `./eval/runs/train/<version>/`.
4. Review the matching local eval artifacts in `./eval/runs/`:
   - `inference/<version>/`
   - `visual-quality/<version>/`
   - `contact-sheets/<version>/`
   - `stability-strips/<version>/`
   - `storyboards/<version>/`
   - `token-agreement/<version>/`
   - `rubric/<version>/`

Agents should use both changelog context and artifact-based eval outputs before concluding whether a model is overall good/bad.
Note: visual eval tools compare predictions against decoded target tokens (`VQ-VAE decode(gt_tokens)`), not raw source RGB frames.

---

## Eval Suite (Step 7)

All tools default to `--device mps`. VQ-VAE checkpoint is always `./vq_vae/checkpoints/vqvae_v2.1.6__epoch100.pt`. Canonical contexts are 4 seeds in `./preprocessedv5/` (defaults built into each tool).

### `diagnostics/inference_diagnostics.py`
Runs 7 scalar diagnostics: entropy, top-k consistency, code distribution, action sensitivity, per-frame quality, VQ-VAE diversity, action transitions. Writes `inference_diagnostics_k{topk}_t{temp}_rd{decay}.json` with a flat `summary` dict (schema_version 2).
```
python diagnostics/inference_diagnostics.py \
    --checkpoint ./checkpoints/<ckpt>.pt \
    --vqvae_ckpt ./vq_vae/checkpoints/vqvae_v2.1.6__epoch100.pt \
    --context_npz ./preprocessedv5/seed_1000_part_1000.npz \
    --output_dir ./eval/runs/inference/<version>/seed_1000/
```

### `eval/eval_tools/contact_sheet_eval.py`
Generates a multi-row PNG per context: decoded target reference / Model AR / Oracle (optional) / token agreement heatmap / edge maps. Companion `metadata.json` has full 20-frame quality curve against the decoded target-token reference.
```
python eval/eval_tools/contact_sheet_eval.py \
    --checkpoint ./checkpoints/<ckpt>.pt \
    --vqvae_ckpt ./vq_vae/checkpoints/vqvae_v2.1.6__epoch100.pt \
    --output_dir ./eval/runs/contact-sheets/<version>/ \
    [--include_oracle]
```

### `eval/eval_tools/stability_strip_eval.py`
Horizontal strip of 20 consecutive frames under a fixed repeated action (static / move_forward / camera_right). Shows flicker and scene maintenance quality. Writes `edge_flicker_l1` and `pred_unique_codes` per frame.
```
python eval/eval_tools/stability_strip_eval.py \
    --checkpoint ./checkpoints/<ckpt>.pt \
    --vqvae_ckpt ./vq_vae/checkpoints/vqvae_v2.1.6__epoch100.pt \
    --output_dir ./eval/runs/stability-strips/<version>/
```

### `eval/eval_tools/transition_storyboard_eval.py`
Storyboard showing last 3 pre-transition + first 8 post-transition frames for 3 hardcoded transitions (static→jump, camera_right→move_forward, camera_right→static). `storyboard_post_au` is annotated on image and is specific to this storyboard protocol.
```
python eval/eval_tools/transition_storyboard_eval.py \
    --checkpoint ./checkpoints/<ckpt>.pt \
    --vqvae_ckpt ./vq_vae/checkpoints/vqvae_v2.1.6__epoch100.pt \
    --output_dir ./eval/runs/storyboards/<version>/
```

### `eval/eval_tools/token_agreement_eval.py`
Per-frame 2×2 panel: predicted RGB / GT RGB / token agreement heatmap / top-1 confidence heatmap. Distinguishes "blurry but structurally correct" from "wrong tokens". Writes `*_token_agreement.metadata.json` with per-frame token/quality metrics.
```
python eval/eval_tools/token_agreement_eval.py \
    --checkpoint ./checkpoints/<ckpt>.pt \
    --vqvae_ckpt ./vq_vae/checkpoints/vqvae_v2.1.6__epoch100.pt \
    --context_npz ./preprocessedv5/seed_1000_part_1000.npz \
    --output_dir ./eval/runs/token-agreement/<version>/seed_1000/
```

### `eval/eval_tools/rubric_summary.py`
Aggregates `inference_diagnostics.json` + contact-sheet metadata + optional stability-strip metadata into a compact rubric with `good/fair/poor` bands. Produces the final agent-readable verdict for a checkpoint.
```
python eval/eval_tools/rubric_summary.py \
    --inference_json ./eval/runs/inference/<version>/seed_1000/inference_diagnostics_k50_t1.0_rd1.0.json \
    --visual_json ./eval/runs/contact-sheets/<version>/seed_1000_part_1000_contact_sheet.metadata.json \
    --stability_json ./eval/runs/stability-strips/<version>/seed_1000_part_1000_stability_strip.metadata.json \
    --output_path ./eval/runs/rubric/<version>/rubric.json
```

---

## Key Scalars (read from JSON summaries)

| Metric | Source key | Good range |
|--------|-----------|------------|
| `static_to_jump_post_au` | `inference_diagnostics.json → summary` | ≥ 40 |
| `avg_max_prob` | `inference_diagnostics.json → summary` | 0.25 – 0.40 |
| `consistency_score` | `inference_diagnostics.json → summary` | ≥ 0.30 |
| `action_effect_magnitude` | `inference_diagnostics.json → summary` | ≥ 0.24 |
| `lpips_mean` | `contact_sheet metadata.json → summary.pred.lpips_mean` | lower is better |
| `edge_flicker_proxy_per_frame` | `stability_strip metadata.json → rows[action=static]` | lower is better |
| `token_accuracy_mean` | `token_agreement metadata.json → summary.token_accuracy_mean` | higher is better |

---

## Model / Data Quick Reference

- **Token grid**: 18×32, codebook size 1024
- **Frame resolution**: 128×72 (VQ-VAE output)
- **Action encoding**: (15,) float32 — yaw(5) + pitch(3) + WASD+jump+sprint+sneak(7)
- **NPZ keys**: `tokens` (N, 18, 32) uint16, `actions` (N-1, 15) float32
- **`load_checkpoint(ckpt, vqvae_ckpt, device)`** returns `(model, vqvae, model_kwargs)` — always 3 values

---
> Source: [peanutChowder/project-ochre](https://github.com/peanutChowder/project-ochre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
