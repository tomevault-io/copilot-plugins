## ctfdex

> This repository is `CTFDex`, a Python-based pipeline for scraping CTF challenges, downloading challenge artifacts, asking Codex to solve them, and submitting discovered flags back to the platform.

# AGENTS.md

## Project Overview

This repository is `CTFDex`, a Python-based pipeline for scraping CTF challenges, downloading challenge artifacts, asking Codex to solve them, and submitting discovered flags back to the platform.

The current code supports two scraping/submission modes:

- `ctfd`: browser automation against a standard CTFd UI with Playwright
- `cyberthon`: GraphQL-based scraping and submission

The main entrypoint is `main.py`.

## Core Execution Flow

Running `python main.py` does the following:

1. Loads YAML config from `config.yaml` or `--config`.
2. Initializes SQLite state in `db/challenges.db`.
3. Scrapes challenge metadata from the configured platform.
4. Creates a per-challenge folder under `challs/<sanitized-name>/`.
5. Downloads challenge files into that folder.
6. Optionally retrieves similar writeups from local embeddings in `embeddings/`.
7. Builds a Codex prompt with challenge text, retrieved examples, and optional `ctf-skills` routing.
8. Runs Codex once or as a small parallel race (`parallel_codex_agents`).
9. Stores prompts and model output in `runs/<sanitized-name>/`.
10. Extracts a flag and submits it through the scraper.
11. Marks successful solves in SQLite.

## Important Files

- `main.py`: orchestration, prompt construction, Codex subprocess execution, retry loop, skill classification
- `scraper.py`: platform-specific scraping and flag submission
- `database.py`: SQLite schema and solved-state tracking
- `retrieval.py`: FAISS + sentence-transformers retrieval over local writeup metadata
- `build_dataset.py`: converts cloned writeup repos into `dataset_index.jsonl`
- `build_embeddings.py`: builds `embeddings/ctf_index.faiss` and `embeddings/ctf_metadata.json`
- `mine_repos.py`: mines/clones public writeup repositories into `dataset/`
- `README.md`: user-facing setup and usage notes

## Data And Generated Artifacts

These paths are working data, not hand-maintained source:

- `challs/`: downloaded challenge files
- `runs/`: per-attempt prompts and Codex output logs
- `db/`: SQLite database
- `dataset/`: mined writeup repositories
- `dataset_index.jsonl`: normalized dataset records
- `embeddings/`: FAISS index and retrieval metadata
- `__pycache__/`: Python bytecode

Do not delete or rewrite these unless the task explicitly requires it.

## Configuration Notes

Configuration lives in YAML files:

- `config.yaml`: active local config
- `config.ctfd.sample.yaml`: sample for CTFd
- `config.cyberthon.sample.yaml`: sample for Cyberthon

`config.yaml` may contain real credentials or competition details. Treat it as sensitive. Prefer editing sample configs or documenting changes without exposing secrets.

Relevant solver controls in `solver`:

- `enable_multiple_codex_agents`
- `parallel_codex_agents`
- `use_ctf_skills`
- `ctf_skills_dir`
- `allow_network_access`
- `allow_package_install`
- `allow_outside_challenge_access`
- `codex_model`
- `codex_version`
- `codex_command`
- `codex_sandbox`
- `codex_add_dirs`

## Dependencies

Main runtime dependencies:

- Python 3
- `playwright`
- `codex` CLI or `npx @openai/codex@<version>`
- `requests`, `PyYAML`
- optional retrieval stack: `sentence-transformers`, `faiss-cpu`

If retrieval assets are missing or embedding deps are unavailable, the solver degrades gracefully and continues without similar-example retrieval.

## Guidance For Agents

- Preserve the current architecture: `main.py` orchestrates, helper modules stay focused.
- Prefer small, local changes over broad refactors.
- Keep platform-specific logic in `scraper.py`, not in the orchestration layer.
- Keep generated directories and large datasets out of edits unless the task is specifically about them.
- Be careful with concurrency changes. `main.py` already uses thread pools at both challenge and agent levels.
- If you modify prompt logic, keep the output contract intact: successful runs should still emit only the flag when possible.
- If you touch retrieval, maintain compatibility with existing `dataset_index.jsonl` and `embeddings/*` conventions.
- If you add config fields, update sample config files and `README.md`.

## Typical Local Commands

```bash
python main.py
python main.py --config config.ctfd.sample.yaml
python build_dataset.py --dataset-dir dataset --output dataset_index.jsonl
python build_embeddings.py --input dataset_index.jsonl
```

## Testing And Validation

There is no obvious comprehensive automated test suite in the current repo. When changing behavior, validate with the narrowest realistic check:

- syntax check relevant Python files
- run the specific helper script you changed
- avoid full scraping or submission unless the task requires live interaction

## Known Repo Characteristics

- The working tree may contain large generated assets and challenge binaries.
- The repo may be intentionally used with permissive Codex sandbox settings.
- `README.md` is brief; this file should help agents orient quickly without reverse-engineering the repo each time.

---
> Source: [Techie-Ernie/ctfdex](https://github.com/Techie-Ernie/ctfdex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
