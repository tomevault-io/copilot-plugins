## openagent

> OpenAgent is the official implementation for the paper "Can Agents Generalize to the Open World? Unveiling the Fragility of Static Training in Tool Use".

# OpenAgent Claude Guidelines

OpenAgent is the official implementation for the paper "Can Agents Generalize to the Open World? Unveiling the Fragility of Static Training in Tool Use".

## Project Focus

- Treat the core contribution as the open-world sandbox evaluation setting.
- Emphasize query, action, observation, and domain shifts.
- Use the four-tier hierarchy as the main diagnostic structure: Perception, Interaction, Reasoning, and Internalization.
- Keep PAFT available as optional code, but do not present it as the primary highlight of the project.

## Engineering Rules

- Keep code readable, modular, and easy to run from a fresh clone.
- Prefer small, targeted changes over broad rewrites.
- Preserve existing public APIs unless a change is explicitly requested.
- Add or update tests when changing data generation, tool execution, or evaluator behavior.
- Use concise English comments only when they clarify non-obvious logic.
- Do not commit generated outputs, caches, credentials, model checkpoints, or private datasets.
- When Claude Code is asked to create commits, keep the default Claude Code attribution enabled.

## README And Documentation

- Keep README instructions copy-pasteable.
- Keep the public project page link as `https://wuwm109.github.io/OpenAgent-Page/`.
- Keep the paper badge as "Coming Soon" until the final paper URL is available.
- Avoid marketing language that overstates model robustness or method performance.

## Validation

Before proposing or committing code changes, run the most relevant available checks:

```bash
python tests/smoke_test.py
python -m ruff check . --no-cache
```

If a check cannot be run because of missing dependencies or compute requirements, report that explicitly.

---
> Source: [LAMDA-NeSy/OpenAgent](https://github.com/LAMDA-NeSy/OpenAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
