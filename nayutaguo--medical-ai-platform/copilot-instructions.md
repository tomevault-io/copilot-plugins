## medical-ai-platform

> This repository is expected to be developed by humans and Codex over multiple sessions. Do not rely on chat history as the source of truth.

# Agent Collaboration Guide

This repository is expected to be developed by humans and Codex over multiple sessions. Do not rely on chat history as the source of truth.

## Required Startup Workflow

Every Codex session must read these files before changing code:

1. `AGENTS.md`
2. `docs/PROJECT_STATUS.md`
3. `docs/ROADMAP.md` and `docs/DECISIONS.md` when the task touches architecture, scope, or dependencies

## Required Shutdown Workflow

Before finishing a work session, update `docs/PROJECT_STATUS.md` with:

- current phase
- completed work
- in-progress work
- unfinished work
- current directory structure
- verified commands
- current environment notes
- important design decisions
- known issues or blockers
- recommended next steps

## Engineering Rules

- Before implementing a non-trivial feature, inspect relevant code in `third_party/` and prefer reuse, adaptation, or porting when license and architecture allow it.
- Keep third-party repositories under `third_party/` only when intentionally cloned.
- Do not copy third-party source into `src/`.
- Record every third-party reference in `docs/THIRD_PARTY.md`.
- If code is copied or substantially adapted, add an ADR and attribution note before or with the code change.
- Use Conda for Python runtime management.
- Use `pyproject.toml` for Python package dependencies.
- Do not add competing dependency sources unless `docs/ENVIRONMENT.md` explains the responsibility split.
- Do not commit `.env`, raw data, processed data, model weights, or generated build outputs.
- Phase 1 uses structured `QuerySpec`, not arbitrary SQL from LLMs.
- SQL generation must use field/table/operator/aggregation allowlists and parameter binding.
- SQL compilation and database execution must remain separate.
- Do not implement arbitrary Python code execution until sandbox limits are designed and documented.
- Do not deploy Hadoop/Spark in phase 1.

## Git Rules

- The main branch is `main`.
- Preserve user changes. Do not reset or revert unrelated work.
- The parent workspace currently contains a read-only `.git/`; the actual repository is initialized in this directory.

---
> Source: [Nayutaguo/medical-ai-platform](https://github.com/Nayutaguo/medical-ai-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
