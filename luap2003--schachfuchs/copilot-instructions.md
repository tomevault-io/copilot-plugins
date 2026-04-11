## schachfuchs

> This file is mandatory reading at the start of every session in this repository.

# AGENTS Guide

## Purpose
This file is mandatory reading at the start of every session in this repository.
It defines how agents should navigate the codebase, implement work, validate changes, and create checkpoints.

## Precedence and Scope
- Scope: applies to the entire repository.
- Override rule: if a deeper-path `AGENTS.md` is added later, it overrides this file for its subtree only.

## Codebase Map
- `lib/core`: engine, opponents, content loading, storage, domain models.
- `lib/features`: feature modules (`home`, `lessons`, `puzzles`, `play`) with screen/bloc/widget code.
- `lib/shared`: reusable UI, board wrapper, theme primitives.
- `assets/content`: content manifest and lesson/puzzle JSON assets.
- `docs`: ADRs and per-service/per-bloc markdown documentation.
- `test`: core tests, feature tests, widget tests, golden smoke tests.

## Task Execution Workflow
1. Explore first: inspect relevant files and current behavior before editing.
2. Implement in small slices.
3. Validate each slice immediately (do not batch all validation at the end).
4. Keep progress steady; avoid large unverified change sets.

## Branch Policy (New Features)
- Do not implement new features directly on `main`.
- Use a feature branch; naming is intentionally flexible.
- Recommended examples:
  - `feature/drag-input`
  - `fix/lesson-flow`
  - `chore/ci-cleanup`
  - `docs/agent-guide`

## Checkpoint Commit Policy
- Create a checkpoint commit after each logical unit is implemented and validated.
- Keep each checkpoint focused and reversible.
- Commit message format:
  - `<type>: <short intent>`
- Add a brief commit body when context is needed (`what changed` and `why`).

## Validation Gate Before Checkpoint Commits
Run all of the following before creating a checkpoint commit:
- `dart format .`
- `flutter analyze`
- `flutter test`

If UI/golden behavior changes:
- update/verify goldens explicitly (for example with `--update-goldens` when intended), and note it in the summary.

## Docs Maintenance Rules
- If core service or bloc behavior changes, update the matching markdown in:
  - `docs/core`
  - `docs/features`
- If architecture or directional decisions change, add/update an ADR in `docs/adr`.
- Example: if `GameBloc` behavior changes, update `docs/features/game_bloc.md` in the same work unit.

## Git Hygiene
- Keep `.gitignore` current for generated and machine-local artifacts.
- Never commit build outputs or local environment files.
- Keep commits small, reviewable, and rollback-friendly.

## Definition of Done (per task)
- Code compiles and runs for the changed area.
- `flutter analyze` passes.
- `flutter test` passes.
- Related docs are updated where behavior changed.
- Final handoff summarizes changed files and next verification steps.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luap2003)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Luap2003)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
