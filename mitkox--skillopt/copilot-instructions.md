## skillopt

> - Treat this repository as a **local-first, open-source SkillOpt fork**.

# Agent instructions for SkillOpt

## Project identity

- Treat this repository as a **local-first, open-source SkillOpt fork**.
- SkillOpt optimizes **skill documents / system prompts**, not model weights.
- Prefer examples and smoke tests that work against a **local OpenAI-compatible backend**.

## Default example workflow

When you need a runnable example, default to the bundled DotNetDebug setup:

- config: `configs/dotnetdebug/local_mitko.yaml`
- sample data: `data/dotnetdebug/tasks.json`
- seed skill: `skillopt/envs/dotnetdebug/skills/initial.md`
- output root: `outputs/dotnetdebug_smoke`

Use small validation runs before suggesting larger experiments. Prefer overrides like:

```bash
--cfg-options \
  train.num_epochs=1 \
  train.batch_size=2 \
  gradient.minibatch_size=2 \
  gradient.analyst_workers=1 \
  env.workers=1 \
  env.limit=2
```

## Documentation expectations

- Put the **local AI path first** in docs and examples.
- Mention cloud backends only as optional alternatives unless the task is explicitly cloud-specific.
- When referring to local inference, prefer `openai_compat` and `http://localhost:8000/v1` unless the user says otherwise.
- Keep the README aligned with `.env.example` and the shipped configs.

## Repo hygiene

- Never commit secrets, `.env` files, private notes, personal scratch files, local outputs, or benchmark dumps.
- Keep private markdown in ignored files such as `*.local.md` or `*.secret.md`.
- Keep IDE state, virtual environments, logs, and generated outputs out of git.
- Before publishing, verify that `.gitignore` still covers local-only files.

## Code and config conventions

- Prefer structured config sections: `model`, `train`, `gradient`, `optimizer`, `env`.
- Reuse repo-native configs instead of inventing ad hoc command lines when possible.
- If adding a new backend, update the backend modules, config mapping, runtime dispatch, docs, and env template together.
- If adding a new benchmark, update the env package, config, CLI entry points, and user-facing docs together.

## Validation

- After code edits, check for errors and run the smallest realistic smoke test when practical.
- If a command fails, report the first concrete failure point and propose the smallest next fix.

---
> Source: [mitkox/SkillOpt](https://github.com/mitkox/SkillOpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
