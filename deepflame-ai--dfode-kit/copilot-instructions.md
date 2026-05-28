## dfode-kit

> This repository contains DFODE-kit, a Python toolkit for sampling combustion states, augmenting and labeling datasets, and training neural-network surrogates for stiff chemistry integration.

# AGENTS.md

This repository contains DFODE-kit, a Python toolkit for sampling combustion states, augmenting and labeling datasets, and training neural-network surrogates for stiff chemistry integration.

## Goals
- Keep changes small, reviewable, and verifiable.
- Optimize for agent legibility and fast iteration.
- Improve experimentation throughput for model architectures, training algorithms, and data pipelines.

## Commands
- Bootstrap lightweight harness env: `make bootstrap-harness`
- Bootstrap docs env: `make bootstrap-docs`
- Activate env: `source .venv/bin/activate`
- Fast syntax check: `make check`
- Run tests: `make test`
- Build docs locally: `make docs-build`
- Full local verification: `make verify`

## Workflow
- Plan before editing for multi-file or ambiguous work.
- Read relevant files before changing them.
- Prefer focused PRs/branches with one main concern each.
- Run `make verify` before proposing a merge.
- If adding a new invariant, encode it in tests or CI.

## Structure
- `dfode_kit/cli/`: CLI entrypoints and subcommands
- `dfode_kit/cases/`: case init, presets, sampling, and DeepFlame/OpenFOAM-facing helpers
- `dfode_kit/data/`: data contracts, HDF5 I/O, and integration helpers
- `dfode_kit/data_operations/`: labeling and augmentation workflows
- `dfode_kit/models/`: model architectures and registries
- `dfode_kit/training/`: training configuration, loops, and preprocessing
- `docs/agents/`: deeper agent-facing docs and workflow conventions
- `tests/`: lightweight harness tests; keep them fast

## Rules
- Do not put large architecture essays in this file; add them under `docs/agents/` and link here.
- Prefer deterministic, non-interactive commands.
- Prefer explicit exceptions over bare asserts for runtime validation changes.
- Keep stdout clean for result output; use logs/errors sparingly.
- Do not mix unrelated refactors in the same branch.
- Do not commit machine-specific local paths to published docs, README examples, or user-facing command snippets. Use portable placeholders like `/path/to/...`, `${VAR}`, or `<worktree-root>` instead.

## Read next
- `docs/agents/README.md`
- `docs/init.md`
- `docs/agents/verification.md`
- `docs/agents/worktrees.md`
- `docs/agents/roadmap.md`
- `docs/agents/package-topology-spec.md`

---
> Source: [deepflame-ai/DFODE-kit](https://github.com/deepflame-ai/DFODE-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
