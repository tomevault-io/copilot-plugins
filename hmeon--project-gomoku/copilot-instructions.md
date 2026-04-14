## project-gomoku

> This repository's primary project is `Battle_Omok_AI/`, a Renju-rule Gomoku AI written in Python.

# Project-Gomoku: Renju Gomoku AI (Battle_Omok_AI)

This repository's primary project is `Battle_Omok_AI/`, a Renju-rule Gomoku AI written in Python.
Historical prototypes live under `reference*/` and are kept unchanged.

## Start Here (Docs)

- `README.md`: overall quickstart, training pipeline, and logging overview
- `Battle_Omok_AI/README.md`: CLI usage and practical option guide
- `Battle_Omok_AI/docs/`: paper-ready docs (architecture/training/logging/rules)
- `PROJECT_REPORT.md`: technical report (paper-style)
- `PATCH_NOTE_v1.md`: changelog

## Core Features

- **Rules:** Renju (Black forbidden moves: 3-3, 4-4, overline; Black wins only with exact five, White with five or more)
- **Search backends:**
  - Iterative deepening minimax (alpha-beta + transposition table + optional VCF)
  - PUCT MCTS (soft pi target support for learning)
- **Learning pipeline:** self-play JSONL generation -> PV (policy-value) training -> iterative auto-train loop with evaluation gating
- **Logging (paper-grade):** CSV metrics under `Battle_Omok_AI/logs/` (`selfplay_metrics.csv`, `train_metrics.csv`, `eval_metrics.csv`)

## Key Entrypoints

- `Battle_Omok_AI/main.py`: match runner (CLI + optional GUI)
- `Battle_Omok_AI/selfplay.py`: self-play data (`*.jsonl`) + stats (`*_stats.json`)
- `Battle_Omok_AI/train_pv.py`: PV training + `train_metrics.csv`
- `Battle_Omok_AI/auto_train.py`: full loop + persistent evaluation logging (`eval_metrics.csv`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hmeon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Hmeon)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
