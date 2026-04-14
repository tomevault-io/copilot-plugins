## botzbotem

> - Root Prolog entry points: `dialog.pl` (bot-to-bot dialog) and `takalbonie.pl` (yes/no mode).

# Repository Guidelines

## Project Structure & Module Organization
- Root Prolog entry points: `dialog.pl` (bot-to-bot dialog) and `takalbonie.pl` (yes/no mode).
- Shared Prolog modules live in `pl/` (API calls, prompts, grammar, dialog logic, output helpers).
- Shell helpers: `bzb.sh` runs `dialog.pl`, `10pytań.sh` runs `takalbonie.pl`.
- Templates and transcripts: `szablon*.txt` are input templates; `10pytań/*.tks` and `rozmowa.tks` store example outputs.
- `CZYTAJMNIE.md` and `README.md` document setup in Polish/English.

## Build, Test, and Development Commands
- `swipl -s dialog.pl` runs the main dialog flow.
- `swipl -s takalbonie.pl` runs the “TakAlboNie” flow.
- `./bzb.sh` and `./10pytań.sh` are Linux shortcuts for the two modes.
- To use a local LLM server, start one of:
  - `llama.cpp/llama-server -m Bielik-11B-v2.3-Instruct.Q4_K_M.gguf --port 8080`
  - `ollama run Bielik-11B-v2.3-Instruct.Q4_K_M`

## Coding Style & Naming Conventions
- Prolog files set UTF-8 explicitly; keep files UTF-8 and preserve Polish identifiers.
- Follow existing predicate style: lowercase names with underscores (e.g., `domyślny_url/1`).
- Keep module declarations and `use_module/consult` directives at the top of each file.
- Prefer short, explicit facts for configuration (see `pl/api.pl`, `dialog.pl`).

## Testing Guidelines
- No automated tests are present. Validate changes manually by running the scripts above.
- When changing grammar or templates, verify parsing and output using sample inputs in `szablon*.txt` and compare with `10pytań/*.tks`.

## Commit & Pull Request Guidelines
- Recent commits use short, descriptive Polish phrases (e.g., “Dodane …”, “Zmiana …”).
- Keep commit messages concise and focused on what changed.
- PRs should include a brief summary, steps to run locally, and any updated templates or output samples.

## Configuration & LLM Notes
- LLM endpoint and model are configured in `pl/api.pl` (`url/1`, `model_bielika/1`).
- Dialog prompts and limits are configured in `dialog.pl` and `pl/api.pl`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PolskiBielik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
