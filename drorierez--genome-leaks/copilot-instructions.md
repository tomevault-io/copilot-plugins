## genome-leaks

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.

## Environment

- The repo is usually used on Windows with PowerShell.
- Work from the repository root:
  `d:\Dropbox\DrorMaster\Thesis\thesis_DNA_gen_ai\code\Genome-Leaks`
- Prefer `rg` for searching files and text.

## Project Context

- This project studies genome generative models and membership/privacy attacks.
- Main attack runner: `attacks/attack_models.py`
- Monte Carlo attack implementation: `attacks/MonteCarlo/MonteCarlo_attack.py`
- Model wrappers live under `models/`.
- Attack outputs are written under `attacks/results/`.

## Validation

For small Python edits, at minimum run:

```powershell
python -m py_compile attacks/attack_models.py attacks/MonteCarlo/MonteCarlo_attack.py
```

Only run the full attack script when explicitly needed. It can be very slow, especially with large Monte Carlo sample counts.

## Performance Notes

- `MONTE_CARLO_N_SAMPLES = 1000000` is expensive even on GPU.
- Monte Carlo work scales roughly with:
  `number_of_candidates * MONTE_CARLO_N_SAMPLES * genome_length`
- Keep GPU distance paths enabled for Euclidean Monte Carlo distances when possible.
- Tune these cautiously:
  - `MONTE_CARLO_GENERATION_BATCH_SIZE`
  - `SYNTHETIC_CACHE_BATCH_SIZE`
  - `CANDIDATE_BATCH_SIZE`
  - `MODEL_GENERATION_BATCH_SIZE`
- If CUDA runs out of memory, reduce `CANDIDATE_BATCH_SIZE` first, then generation/cache batch sizes.

## Output Safety

- Do not delete files in `attacks/results/` unless explicitly asked.
- Prediction CSVs, metric CSVs, ROC PNGs, and synthetic caches may be useful experiment artifacts.
- Avoid changing default sample counts or model-selection regexes unless the user asks.

## Style

- Prefer small, explicit edits over broad refactors.
- Keep experiment code readable and easy to adjust.
- Preserve existing file paths and result naming conventions unless there is a clear reason to change them.

---
> Source: [DroriErez/Genome-Leaks](https://github.com/DroriErez/Genome-Leaks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
