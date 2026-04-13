## turboquant-lab

> Implementation of TurboQuant-style vector quantization for studying compression vs accuracy trade-offs in high-dimensional vectors (relevant to LLM KV cache).

# MiniTurboQuant Lab

Implementation of TurboQuant-style vector quantization for studying compression vs accuracy trade-offs in high-dimensional vectors (relevant to LLM KV cache).

## Project Goal

Build a modular pipeline that compresses vectors while preserving inner product accuracy, demonstrating why naive MSE quantization fails and how residual correction fixes it.

## Stack

- **Backend**: Python, NumPy / PyTorch CPU
- **Frontend**: React, Tailwind CSS, Recharts
- **Constraints**: CPU only (16GB RAM), fast execution, fully reproducible (fixed seeds)

## Architecture: Two-Stage Pipeline

```
x → Normalize → Rotate (Π) → Scalar Quantize → Reconstruct (x_mse)
                                                      ↓
                                               Residual r = x - x_mse
                                                      ↓
                                               QJL: z = sign(S·r), x_qjl = sqrt(π/2)/d · ||r||₂ · Sᵀz
                                                      ↓
                                          Final: x_tilde = x_mse + x_qjl
```

### Key Ideas
1. **Random rotation** (QR decomp → orthogonal Π): makes coordinates ~independent → enables scalar quantization
2. **Scalar quantization (MSE stage)**: quantize each rotated coordinate independently; b ∈ {1,2,3,4} bits
3. **Residual + QJL correction**: 1-bit random projection of residual; restores unbiased inner product estimates

## Implementation Tasks (T1 → T12)

- T1: Dataset generation (Gaussian vectors, unit-norm normalized)
- T2: Rotation module (random orthogonal matrix via QR)
- T3: Scalar quantizer (uniform codebook, assign nearest centroid)
- T4: MSE reconstruction (inverse rotation)
- T5: Residual computation
- T6: QJL module (1-bit projection + reconstruction)
- T7: Full TurboQuant pipeline
- T8: Evaluation metrics (MSE, cosine similarity, IP error, bias)
- T9: Experiment runner (sweep over b, d, seeds)
- T10: Logging (JSON/CSV)
- T11: Visualization (plots: MSE vs bits, IP error vs bits, bias vs bits)
- T12: Results summary

## Experiment Parameters

- Dimensions: `d ∈ {64, 128, 256, 512, 1024}`
- Bit-widths: `b ∈ {1, 2, 3, 4}`
- Multiple seeds per config
- Compare: MSE-only vs full TurboQuant

## Correct Interpretation of Results

### Key distinction: bias vs variance

The old framing "TurboQuant reduces IP error" is **misleading at low dimensions**.
The correct framing:

**MSE-only:** introduces a **multiplicative bias** `E[⟨y,x̃⟩] = α·⟨y,x⟩` where `α < 1`
- b=1: α ≈ 2/π ≈ 0.637 (proven in paper, confirmed empirically)
- b→∞: α → 1 but never exactly 1
- Bias is **dimension-independent** — does NOT improve with d

**TurboQuant:** **unbiased** by construction — `E[⟨y,x̃⟩] = ⟨y,x⟩` exactly (α = 1.0)
- But QJL adds variance: `Var ≤ π/(2d) · ‖r‖²`
- Variance **decreases with d** — at LLM scale (d=1024), variance is negligible

### The right evaluation metric

`ip_error = mean((ê-t)²)` mixes bias and variance — **misleading alone**.

The correct key metric is the **bias coefficient** α:
```
α = Cov(⟨y,x̃⟩, ⟨y,x⟩) / Var(⟨y,x⟩)
```
- MSE-only b=1, any d: α ≈ 0.637
- TurboQuant, any b, any d: α ≈ 1.000

### When does TurboQuant win?

At **high dimensions** (d ≥ 256): TurboQuant's variance shrinks proportionally to 1/d
while MSE-only's bias is constant. This is exactly the LLM KV-cache regime.

At **low dimensions** (d = 64): TurboQuant's variance may dominate, making its total
IP error higher — but it is still UNBIASED, which is the correct property for many applications.

### Plots generated

| Plot | What it shows |
|------|---------------|
| `mse_vs_bits.png` | Reconstruction MSE (TurboQuant higher — expected) |
| `bias_coef_vs_bits.png` | **KEY**: α ≈ 0.637 for MSE-only b=1, α ≈ 1.0 for TurboQuant |
| `ip_variance_vs_bits.png` | TurboQuant has higher variance; variance shrinks with d |
| `ip_error_vs_bits.png` | Total IP MSE (mixes bias+variance) |
| `scaling_with_d.png` | Shows TurboQuant advantage grows with d |

## Frontend

Narrative-driven research explainer (React + Vite + Tailwind + Recharts).
Located in `frontend/`. Run with `npm run dev` from that directory.

### Pages

| Page | Route | Purpose |
|------|-------|---------|
| The Story | `/` | Scroll narrative: problem → hidden failure → TurboQuant solution → proof |
| Explorer | `/results` | Interactive: dimension/bit-width selectors + live Recharts charts |
| Insights | `/insights` | 4 key insights with comparisons and the final takeaway |

### Key design choices
- Dark "observatory" theme (#04080f background)
- Fonts: Outfit (display) + JetBrains Mono (numbers) + DM Sans (body)
- MSE-only = blue (#3b82f6), TurboQuant = orange (#f97316)
- The **bias_coef chart** is the centrepiece — shows α≈0.637 vs α≈1.000
- Scroll-reveal animations on Home sections
- Dynamic charts pull from `public/data/results.json`

### Running
```bash
# Re-generate experiment results
uv run python run_experiments.py

# Copy updated assets to frontend
cp results/*.png frontend/public/images/
cp results/results.json frontend/public/data/

# Start dev server
cd frontend && npm run dev
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thejasrao262003)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thejasrao262003)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
