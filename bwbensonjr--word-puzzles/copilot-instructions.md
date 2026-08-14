## word-puzzles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Tools for working on word puzzles (currently Wordle). Two independent subprojects:

- `wordle-helper/` — A constraint-based candidate-word finder, implemented twice: a static browser app (`index.html`, Vue 3 + Tailwind via CDN, no build step) and a Python CLI (`wordle.py`). Both share the same algorithm and the `word-list-5.txt` dictionary.
- `wordle-research/` — A `uv`-managed Python project for analyzing the history of past Wordle solutions.

## Commands

### wordle-helper

```bash
# Regenerate the 5-letter dictionary from the system dictionary
cd wordle-helper && make            # writes word-list-5.txt from /usr/share/dict/words

# Run the CLI: <pattern> [out_letters] [in_letters]
# pattern is a 5-char regex body, e.g. ".a..e" or "[^xq]...."
python wordle.py "s..re" aeiou st

# Run the browser app (must be served, not opened as file:// — it fetches word-list-5.txt)
python -m http.server 8000         # then open http://localhost:8000/index.html
```

### wordle-research

```bash
cd wordle-research
uv sync                             # install dependencies into .venv
uv run python wordle_history.py     # module currently exposes functions; no main()

# Download personal NYT Wordle history (needs the NYT-S cookie; see README)
uv run python nyt_history.py extract-cookie   # after logging in via webctl
uv run python nyt_history.py download         # -> data/wordle_personal_history.csv
```

## Architecture notes

- **Shared algorithm.** Both `wordle-helper` implementations do the same three steps: (1) filter the dictionary by a positional regex pattern plus include/exclude letter sets, (2) build per-position letter-frequency counters over the *surviving* candidates (`calculate_weights` / `calculateWeights`), (3) score each word by summing those position frequencies. Scores are therefore relative to the current candidate set, not absolute. The CLI sorts ascending (best last); the browser app sorts descending (best first).

- **Constraint model in the browser app.** Each of the 5 positions is `disallow` (regex `[^...]`) or `require` (regex `[...]`). "Must include" letters are *derived* from the disallow inputs (`computedInLetters`) on the theory that a letter you marked yellow somewhere must be in the word — there is no separate include field in the UI.

- **`wordle-research` data source.** Past solutions are fetched live from `HISTORY_URL` (an S3-hosted `history.csv`); `score_word`/`letter_freqs` there divide by word/count length to produce normalized frequencies, unlike the helper's raw counts.

- `word-list-5.txt` is a generated artifact (see the Makefile) but is checked in, since the browser app fetches it at runtime.

- **`nyt_history.py` (personal history).** NYT puzzle `id` is **not** sequential by date (assigned from a non-chronological editorial pool), so it can't be computed — `nyt_history.py` looks up each date's `id` and `solution` from the public `svc/wordle/v2/<date>.json` endpoint (cached in `data/puzzle_index.json`), then fetches the player's saved boards from the authenticated `latests?puzzle_ids=...` endpoint using the `NYT-S` cookie. The cookie is obtained by logging in via `webctl` and extracted from the saved webctl session profile; `data/` and `.nyt_cookie` are gitignored secrets/artifacts.

## Conventions

- Python: manage `wordle-research` with `uv` (not `pip`); use double quotes for strings.
- `*~` editor backup files are gitignored; do not commit or edit them.

---
> Source: [bwbensonjr/word-puzzles](https://github.com/bwbensonjr/word-puzzles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
