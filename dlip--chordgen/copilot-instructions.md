## chordgen

> Minimal guidance for agents working in this repo.

# AGENTS.md

Minimal guidance for agents working in this repo.

## Project layout

- `src/chordgen/` — Python package
  - `cli.py` — Typer entry point (`setup`, `gen`, `output`)
  - `vocab/` — frequency-list sources (`subtlex.py`) and CSV pipeline
  - `gen.py` — top-level pipeline: score → alts → assign → write
  - `scorer.py` — generates and scores candidate chord options per word
  - `alt_generator.py` — category/inflector registry for alt slots
  - `assigner.py` — smart-greedy + 2-swap chord assignment
  - `output/` — emitters (qmk, zmk, kanata, charachorder, training, learn)
  - `keyboards/` — standard and directional keyboard models
  - `config.py` — pydantic schema for `~/.config/chordgen/config.yaml`
  - `chord.py` — `Chord` TypedDict (the row shape of `chords.csv`)
- `docs/schema.md` — generated config schema docs

## Conventions

- Python 3.11+. Managed with `uv`.
- Edit existing files in preference to creating new ones.
- Do not add `config.yaml` knobs casually; the user manages chords.csv by hand
  after `setup`.
- The `frequency` column in `chords.csv` is source-defined (Zipf 0–7 for
  SUBTLEX). Don't assume Zipf in code — treat it as a float.
- Alt generation must run before chord assignment in `gen.py` so the assigner
  can see alt-coverage relationships.
- New functionality should include unit tests
- When the user says 'shipit' you should:
  - run `uv run chordgen schema`
  - Update docs/changelog.md
      - Find the heading matching the version string in pyproject.toml.
      - If no such heading exists in CHANGELOG.md, add it at the top.
      - Append new entries as bullet points under that heading. Do NOT
        create a heading for a different version or remove existing
        entries.
  - Update user docs in /docs
  - Ensure unit tests pass
  - Create a git commit with a brief summary
  - Push the changes

## Common commands

```sh
uv run chordgen setup --force   # regenerate config.yaml + chords.csv
uv run chordgen gen             # assign chords + alts
uv run chordgen output          # write firmware/training files
```

---
> Source: [dlip/chordgen](https://github.com/dlip/chordgen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
