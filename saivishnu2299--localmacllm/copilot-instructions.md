## project-structure

> Use these anchors to navigate the repository and understand where things live.

# Project Structure and Key Entry Points

Use these anchors to navigate the repository and understand where things live.

- PRD: [docs/LocalMacLLM-prd.md](mdc:docs/LocalMacLLM-prd.md)
- Tasks: [tasks/tasks-LocalMacLLM-prd.md](mdc:tasks/tasks-LocalMacLLM-prd.md)
- Setup & usage: [README.md](mdc:README.md)
- Dependencies: [requirements.txt](mdc:requirements.txt)
- Ignore rules: [.gitignore](mdc:.gitignore)

## Rules and Guides

- PRD creation guide: [.cursor/rules/create-prd.md](mdc:.cursor/rules/create-prd.md)
- Task list generation guide: [.cursor/rules/generate-tasks.md](mdc:.cursor/rules/generate-tasks.md)
- Task list process guide: [.cursor/rules/process-task-list.md](mdc:.cursor/rules/process-task-list.md)

## Implemented code

- TinyStories downloader and splitter: [src/data/download_tinystories.py](mdc:src/data/download_tinystories.py)

## Planned (to be created during tasks)

- Tokenizer trainer: [src/tokenizer/train_sentencepiece.py](mdc:src/tokenizer/train_sentencepiece.py)
- Preprocessing/packing: [src/preprocess/pack_sequences.py](mdc:src/preprocess/pack_sequences.py)
- Dataset + loaders: [src/datasets/tinystories_dataset.py](mdc:src/datasets/tinystories_dataset.py)
- GPT model (MLX): [src/model/gpt_mlx.py](mdc:src/model/gpt_mlx.py)
- Train/Eval/Generate CLIs: [src/train.py](mdc:src/train.py), [src/eval.py](mdc:src/eval.py), [src/generate.py](mdc:src/generate.py)
- Quick script: [scripts/run_quick_train.sh](mdc:scripts/run_quick_train.sh)

## Conventions

- Artifacts under `artifacts/` (tokenizer, checkpoints)
- Data under `data/` (raw and packed)
- Source under `src/` by area (data, tokenizer, preprocess, datasets, model)

---
> Source: [saivishnu2299/LocalMacLLM](https://github.com/saivishnu2299/LocalMacLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
