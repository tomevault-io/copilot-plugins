## multimodaljepa

> You are in a git worktree on an `autoresearch/...` branch unless the operator says otherwise.

# Multimodal autoresearch (Gemini CLI)

You are in a git worktree on an `autoresearch/...` branch unless the operator says otherwise.

@./training/program.md

## Extra constraints (match Claude/OpenCode workers)

- Never use `git commit -a` or `git reset --hard`.
- Stage only explicitly changed research files; restore only allowed files when reverting a bad experiment.
- Editable research files: `training/lejepa_multimodal.py`, `training/run.py`, `training/results.tsv` (see `training/program.md`).
- Prefer embedding quality and interpretable changes; `EVAL_SCORE` is the final judge.

## Optional subagent

For a dedicated specialist session, the repo defines `@autoresearch` (see `.gemini/agents/autoresearch.md`).

---
> Source: [AdamGabet/MultiModalJepa](https://github.com/AdamGabet/MultiModalJepa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
