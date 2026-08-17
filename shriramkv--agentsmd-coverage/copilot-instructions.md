## agentsmd-coverage

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this is

A tool that scans a repository and reports AGENTS.md coverage: which code
directories and package boundaries have a governing AGENTS.md, and which do not.
Pure Python, standard library only. No network access. Read only against the
target repo.

## Layout

- `agentsmd_coverage/scan.py` walks the tree and classifies directories as
  significant (enough source) or package boundaries (a build manifest present),
  and records where AGENTS.md files live.
- `agentsmd_coverage/coverage.py` computes the coverage model, findings, and the
  A-F score from three components: root presence, inherited coverage, and local
  coverage at package boundaries.
- `agentsmd_coverage/report.py` renders console, JSON, Markdown, and badge output.
- `agentsmd_coverage/cli.py` is the argparse entry point and CI gates.
- `tests/fixtures/` holds two deliberate repo trees, one well covered and one with
  gaps. Do not add a root AGENTS.md inside `fixtures/gaps`, its absence is tested.

## Rules for changes

- Standard library only. No third party dependencies.
- New classification logic goes in `scan.py`, new scoring logic in `coverage.py`.
- Keep directory ignores conservative and well known (see DEFAULT_IGNORES).
- Do not use em dashes in prose or comments.
- Run `python3 -m unittest discover -s tests` before finishing.

---
> Source: [shriramkv/agentsmd-coverage](https://github.com/shriramkv/agentsmd-coverage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
