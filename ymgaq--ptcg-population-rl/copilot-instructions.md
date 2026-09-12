## ptcg-population-rl

> This repository is a minimal implementation accompanying the Pokémon TCG AI Battle writeup.

# AGENTS.md

## Purpose

This repository is a minimal implementation accompanying the Pokémon TCG AI Battle writeup.
It connects a C++ arena, GBDT training/inference, Transformer distillation/inference, and PPO.
It serves human developers and coding agents; prioritize clarity and reproducibility.

## Scope and implementation order

1. Establish external engine setup and the C++ match interface.
2. Run and record matches with the four official decks and simple policies.
3. Connect GBDT training and C++ inference through one observation/legal-action contract.
4. Distill teacher matches into a Transformer and compare Python with C++ inference.
5. Complete an on-policy rollout, GAE, PPO update, and weight export cycle.

Avoid large operational systems and generic frameworks.
Matchup experts, rating websites, and cloud CI/CD require a separate request. Local rating measurement is in scope.
An independent implementation of the game rules is outside the initial scope.

## License and external assets

- Release repository code, documentation, and figures under Apache-2.0. `LICENSE` is authoritative; separately downloaded assets retain their upstream terms.
- This is an independent Git repository. Do not modify the original competition repository.
- Do not copy entire agents or operational scripts indiscriminately when consulting original code.
- Port only what is needed and preserve third-party copyright and license notices.
- Do not add credentials, personal paths, private replays, submission binaries, or trained weights without authorization.
- Document how to obtain and install required assets that cannot be redistributed.

## Training and inference contracts

- Bind initial card IDs and counts to model assets; fail explicitly on mismatch.
- Align features, vocabularies, option order, masks, multiple selections, and STOP between training and inference.
- Use only information observable to the acting player; never use an omniscient spectator state as model input.
- Pair each replay action with the observation before that selection.
- Split training and validation data by complete games.
- PPO rollouts must identify the collection policy/weights and retain old log probabilities, Value, and terminal outcomes.
- Make rewards, GAE, clipping, entropy, update budget, and opponent distribution explicit configuration.

## Validation

- Keep a minimal working example and instructions for each new feature.
- Compare Python and C++ features, outputs, and actions on identical GBDT/Transformer inputs, with explicit floating-point tolerances.
- Record agent/engine errors and include self-play in match validation.
- Validate PPO collection, updates, export, and matches with the updated policy.
- Do not treat a few successful matches as evidence of stronger play. Strength comparisons require a fixed baseline, common seeds/seats/opponents, uncertainty estimates, and checks for matchup regressions.
- Report hardware, thread count, input size, and timing scope with performance measurements.

## Development and Git

- Keep `src/` divided into `agents/`, `engine/`, and `tools/`. The engine owns observations, legal actions, match progression, and results; it must not implement model-specific features, inference, training, or weight loading.
- Keep each trainable agent's Python implementations in `src/agents/<agent>/{gbdt,transformer,ppo}/`, with method-specific C++ in `cpp/` and commands in `scripts/`. Do not create per-agent READMEs; document usage in the root README and CLI `--help`. Keep cross-method helpers inside the same agent, not a shared directory such as `src/gbdt/`.
- Agent-level `cpp/` contains rule policies, factories, and training records; `scripts/` contains cross-method teacher collection; `tests/` contains agent-specific contract tests. Track `deck.csv` in Git and keep the official Python policy in `main.py`.
- `src/agents/cpp/registry.h` only registers and connects agents. Add learning methods through agent factories and recording hooks, not model-specific arena branches.
- `src/tools/` contains asset downloads, whole-project builds, official-policy validation, and model-independent ratings. Agent-specific training, extraction, export, and validation commands belong inside the agent directory.
- Download the competition engine into ignored `src/engine/cg/`. Include the unmodified files from repository-owned `src/engine/cpp/` code. Do not patch distribution files to hide defects.
- Work directly on `main`; commit and push to `main` after each task's necessary validation and diff review. Never create or switch branches without explicit user instructions. Keep unrelated changes out of the same commit.
- Create PRs only when explicitly requested. Never overwrite someone else's uncommitted work.
- Exclude datasets, build artifacts, checkpoints, and logs from Git.
- Use only relative repository-local paths in `.gitignore`; do not add leading `/`, personal absolute paths, or paths containing `../`.
- Document only dependencies and commands that have been checked. Distinguish implemented, planned, and unverified behavior; never invent features or measurements.
- Keep data/output paths configurable; do not require a particular Slurm installation or personal absolute path.
- Format Python with Ruff and annotate function inputs/returns. Use English docstrings to explain purpose and relevant contracts.
- Write READMEs and code comments in English. Maintain `README.ja.md` as the sole Japanese README, linked from the beginning of the root README summary.
- Use Japanese card names in Japanese prose. Refer to competition leaderboard ratings as “LB ratings” in English and “LBレーティング” in Japanese.
- Public documentation must be understandable from the public writeup and this repository alone. Do not rely on inaccessible repository nicknames or local paths.
- Describe bounded checks as “validation” in English and “動作確認” in Japanese.

---
> Source: [ymgaq/ptcg-population-rl](https://github.com/ymgaq/ptcg-population-rl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
