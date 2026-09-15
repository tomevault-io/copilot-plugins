## devflow-project-skeleton-v13-fresh

> This repository is a fresh DevFlow-managed project template. Claude Code runtime instructions are under `.claude/`. The human-editable DevFlow control plane is under `devflow/`.

# DevFlow Project Instructions

This repository is a fresh DevFlow-managed project template. Claude Code runtime instructions are under `.claude/`. The human-editable DevFlow control plane is under `devflow/`.

## Project discovery

This repository intentionally contains no preconfigured product, client, application, technology, architecture, deployment, or business details.

Before making project-specific decisions:
1. inspect the supplied source material and existing repository;
2. discover the repository and application structure;
3. record confirmed findings in the DevFlow control plane;
4. mark unresolved material information as `UNKNOWN` and create an Open Question when appropriate.

Do not invent application boundaries or assume a particular monorepo layout. If the project is a monorepo, discover its applications and packages from the repository or approved project sources.

## Mandatory no-assumption behaviour

Do not invent or silently assume material requirements, technology choices, versions, security behaviour, permissions, data behaviour, design behaviour, business rules, application boundaries, or deployment boundaries. Check approved project knowledge first. If unresolved and material, create/log an Open Question under `devflow/questions/`.

## Source and control plane

- Original project source material: `client-requirements/`
- DevFlow records/state: `devflow/`
- Claude runtime: `.claude/`
- Design references: `devflow/design/` and project-provided design references
- Technical decisions: `devflow/technical/`

The Source of Truth remains authoritative. Registries normalize and link; they do not silently override the Source of Truth.

## Design authority

If approved design material exists for a page or flow, use it as the visual authority. If no approved design exists, inspect approved project patterns and record the chosen fallback. Do not silently invent a new visual language.

## Development gate

No dependent implementation may proceed through a blocking unresolved Open Question, missing mandatory approval, or explicitly deferred requirement.

Tests, screenshots, visual testing, CI/CD and deployment controls are configuration-controlled and must never be reported as executed merely because they are enabled.

See `.claude/CLAUDE.md` for the complete Claude Code runtime contract.

---
> Source: [zuber-surya/DevFlow-Project-Skeleton-v13-Fresh](https://github.com/zuber-surya/DevFlow-Project-Skeleton-v13-Fresh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
