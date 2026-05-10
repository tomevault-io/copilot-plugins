## hpc-skills

> Use `skills/` as the canonical portable skill collection.

# Repository Guidance

Use `skills/` as the canonical portable skill collection.

## Working rules

- Start with the target skill's `SKILL.md`.
- Load only the specific files in `references/` that match the task.
- Keep `SKILL.md` concise and move deep domain knowledge into `references/`.
- Keep skill names stable and lowercase with hyphens only.
- Prefer official upstream documentation when enriching technical knowledge.

## Repository split

- `skills/` contains the canonical installable skill packages.

## Canonical Skill Shape

Every maintained skill in `skills/` should follow the same top-level shape:

- `SKILL.md`
- `agents/openai.yaml`
- `references/`
- `assets/templates/` when reusable scaffolds exist

Repository maintenance should converge toward this shape instead of introducing one-off layouts.

When a maintained skill needs executable shared tooling, keep it inside an optional `scripts/` directory within that skill instead of creating alternate top-level layouts.

---
> Source: [SciMate-AI/HPC-Skills](https://github.com/SciMate-AI/HPC-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
