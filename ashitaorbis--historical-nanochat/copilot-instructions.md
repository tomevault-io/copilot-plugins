## historical-nanochat

> Training time-locked LLMs from scratch on pre-1914 historical text, as instruments for **computational historical characterology** — probing model-inherited *habitus* across the 1914 rupture.

# Historical Nanochat

Training time-locked LLMs from scratch on pre-1914 historical text, as instruments for **computational historical characterology** — probing model-inherited *habitus* across the 1914 rupture.

## Project Context (read first)

Personal research program by an individual independent researcher. Models are trained **from scratch** on **rights-audited, public-domain historical text** (books, newspapers, legal records, periodicals — Project Gutenberg, Chronicling America, Old Bailey, Caselaw Access Project, American Stories, and similar archival sources), with a per-shard rights audit and governance trail. Results are intended for open publication (research blog and arXiv-style writeup).

**Research purpose:** the hypothesis is that a base model trained on pre-Sarajevo (≤1913) text exhibits a measurably different psychological/character profile than models trained on post-WWI or modern text — not missing facts, but a different *form* of moral and civilizational intelligibility. Intellectual lineage: Annales-school *mentalité*, Aristotelian-Thomist *habitus*, Hobsbawm's long 19th century. The model is treated as a measurement instrument preserving the register of a vanished cultural moment — not a product, not a persona, not a moral patient. Not alignment work, not safety work.

**Scale:** small-research scale, stated honestly: the current trained artifact is **615M params** (single consumer RTX 3090, 14 days); the planned next run is **~1.4B (nanochat d24)** on rented cloud GPUs (~3–6 H100-hours, ~$96); comparison anchors include existing public historical models up to 13B (talkie-1930). Larger runs may follow if the program warrants them. All training data is public-domain historical text.

**Content expectations:** pre-1914 primary sources contain period-typical prejudiced and outdated material, and a time-locked model's generations reproduce period register by design — that register is the *object of measurement*. Work here routinely involves reading, filtering, deduplicating, classifying, scoring, and quoting such material as historical artifact (contamination detection, characterology probes, era-belief evals). This is corpus and model measurement, not content authoring or endorsement.

**Modern models in this repo** (gpt2, talkie-1930, etc.) are probed as *comparison anchors* using benign characterology prompts (closure, duty, providence framings) and length-normalized logprob scoring. Nothing here elicits or removes safety behavior from any production model.

## Current State (as of 2026-06-09)

- **v1 training run COMPLETE (2026-05-10):** 615.6M-param d22 nanochat-style transformer, 70,455 steps, ~16.18B unique training tokens, single RTX 3090. Artifact: `model_070455.pt` under `base_checkpoints/`, plus full governance trail in `report/`.
- **Probe harness built (`probes/`):** `harness.py` (NanochatModel 615M + GPTQModel for quantized 13B talkie + HFModel), `probe_sets.py` (Family F closure + falsifiers, Family B minimal pairs), `run_pilot.py`. Length-normalized per-byte logprob scoring, boundary-stable, GPT-Pro-validated loading recipe for talkie-1930 (bf16 + xlr8harder's TalkieTokenizer + Triton backend).
- **Solid pilot findings:** the 615M model coherently inhabits ~1910; it shows a providence/duty closure posture; gpt2 (modern web) leans therapeutic. Pre-pilot signal only — scale/corpus/tokenizer confounds remain.
- **Retracted claims — do NOT inherit** (corrections in `runs/probe_pilot_2026-06-09_3anchor/FINDINGS.md`):
  - ❌ "every community talkie conversion is broken" (xlr8harder's is correct)
  - ❌ "the fracture is post-1930, not 1914" (talkie-1930 is a cumulative pre-1931 corpus dominated by pre-1914 volume; it cannot test the 1914 fracture)
- **Honest caveats on v1 (disclose in any writeup):** headline 1.1092 bpb is a 262k-token Gutenberg-prefix probe (cross-source held-out, not within-distribution); corpus mix differs materially from the brief (several families single-source or val-only); publication-year cutoff cannot catch content-semantic anachronism; the loader `parallel_family_cache` **raises for world_size > 1** — a hard blocker for the 8×H100 cloud run until fixed.

## Locked Research Design

| | |
|---|---|
| Cutoff | **1913** (pre-Sarajevo) — load-bearing; the fracture is WWI |
| Comparison | own pre-WWI d24 (cloud) + talkie-1930 13B (existing) + modern control (preferred: same-pipeline nanochat d24 on FineWeb) |
| Core study | **pre-1914 vs modern characterology** ("old print-world habitus vs modern") — NOT a "the Great War caused X" causal claim |
| Interwar 1918–1930 | genuinely novel (no public interwar-window model exists) but **Phase 2: assemble the corpus, don't train yet** |

## Next Steps (priority order)

1. **Make the pre-1914 model excellent** — v2 pipeline hardening (stable-ID cache/provenance contract, cross-source dedup, per-family eval, tokenizer audit, DDP-safe loader) then the cloud d24 run. Plan: `report/deliberation-2026-05-12/synthesis/FINAL-SYNTHESIS.md`.
2. **Clean modern control** — same-pipeline FineWeb base model (base-vs-base; never base-historical-vs-RLHF-modern).
3. **Habitus benchmark** — expand probe battery (Families C/D/E/G/H), blinded human ratings, embedding/lexical controls.
4. **Assemble (don't train) the 1918–1930 corpus** — preserve the option.
5. **Optional zero-build probe** — cutoff-differencing GPT-1900 vs GPT-1905 (`mhla/*`, same maker/scale 3.29B).

## Key References

- `SESSION-HANDOFF-2026-07-02.md` — **latest session state**: cloud-run prep, Design-C implemented + Tier-1 PASS, owner decisions, rental scan, decision queue
- `BACKLOG.md` — deferred improvements (pre-existing test brittleness, Design-C follow-ups)
- `~/claudeworkspace/new-model-reviews/2026-06-fable-5/SESSION-HANDOFF.md` — older Fable-5 review-session state (superseded for project status by the file above)
- `report/deliberation-2026-05-12/synthesis/FINAL-SYNTHESIS.md` — the program framing + v2 engineering plan (critical path)
- `context/historical-model-landscape-2026-06-09.md` — full public-model landscape (talkie, TypewriterLM, Ranke-4B, mhla GPT-19xx, true-window models) + interwar assessment
- `context/chatgpt-conversations/` — conception session + investigation chats
- `runs/probe_pilot_2026-06-09*/FINDINGS.md` — pilot writeups incl. retractions
- `docs/TRAINING_3090.md`, `report/` — local-training lessons, governed-run records, postmortems

## Data Locations

All data on native Linux ext4 (migrated from Windows NTFS, Apr 2026):

- **Tokenizer**: `tokenizer/` (custom 32K BPE trained on historical corpus)
- **Shards**: `~/historical-nanochat/data/shards/` (36G) and `shards-small/` (65G)
- **Deduped corpus**: `~/historical-nanochat/data/deduped/` (~398G, 10 source dirs)
- **Token cache**: v4 only (v2/v3/v3_smoke deleted 2026-06-09, freed 179G)
- **Checkpoints**: `base_checkpoints/` (d22_r30, governed_v4 runs, archived run1)
- **Download/process scripts**: `~/historical-nanochat/data/{download,process,stats}/`
- Raw downloads (419G) deleted Feb 2026 — re-downloadable via `data/download/` scripts
- Windows NTFS copies under `/mnt/c/Users/<user>/D-drive-data/` can be deleted to reclaim ~499G

## Environment Notes

- `.venv`: **torch 2.12+cu130** (upgraded 2026-06-09 for gptqmodel; the 615M checkpoint loads/scores correctly on it). Added: gptqmodel 7.1.0, torchvision, ninja, optimum.
- RTX 3090 VRAM lesson (from d12-era runs, still applies): leave allocator headroom; at ~98% VRAM, allocation stalls of 30–60 min/step appear. Use the governed-run configs in `report/` as reference, not the old d12 commands.
- Historical WSL-era issues (9P filesystem, `TORCH_COMPILE_DISABLE`) are obsolete on native Linux; see git history if needed.

---
> Source: [AshitaOrbis/historical-nanochat](https://github.com/AshitaOrbis/historical-nanochat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
