## ironmandiscordreply

> - When communicating your results back to me, explain what you did and what happened in plain, clear English. Avoid jargon, technical implementation details, and code-speak in your final responses. Write as if you're explaining to a smart person who isn't looking at the code. Your actual work (how you think, plan, write code, debug, and solve problems) should stay fully technical and rigorous. This only applies to how you talk to me about it.

# Repository Guidelines

## Reporting and Verification
- When communicating your results back to me, explain what you did and what happened in plain, clear English. Avoid jargon, technical implementation details, and code-speak in your final responses. Write as if you're explaining to a smart person who isn't looking at the code. Your actual work (how you think, plan, write code, debug, and solve problems) should stay fully technical and rigorous. This only applies to how you talk to me about it.

- Before reporting back to me, if at all possible, verify your own work. Don't just write code and assume it's done. Actually test it using the tools available to you. If possible, run it, check the output, and confirm it does what was asked. If you're building something visual like a web app, view the pages, click through the flows, and check that things render and behave correctly. If you're writing a script, run it against real or representative input and inspect the results. If there are edge cases you can simulate, try them.

- Define finishing criteria for yourself before you start: what does "done" look like for this task? Use that as your checklist before you come back to me. If something fails or looks off, fix it and re-test. Don't just flag it and hand it back. The goal is to keep me out of the loop on iteration. I want to receive finished, working results, not a first draft that needs me to spot-check it. Only come back to me when you've confirmed things work, or when you've genuinely hit a wall that requires my input.

## Project Structure & Module Organization
- `src/`: core Python modules such as `gui.py`, `discord_client.py`, and `main.py`.
- `config/`: runtime configuration (`config.json`, `example_config.json`).
- `assets/`: UI assets; `logs/`: runtime logs and debug output.
- Top-level scripts: `run.py` (launch the app) and `build.py` plus `*.spec` (PyInstaller packaging).

## Build, Test, and Development Commands
- `pip install -r requirements.txt`: install Python dependencies.
- `python run.py`: start the desktop GUI locally.
- `python build.py --target windows` or `python build.py --target mac`: build platform packages with PyInstaller.

## Coding Style & Naming Conventions
Use Python 3 with 4-space indentation. Keep files, functions, and variables in `snake_case`, classes in `PascalCase`, and constants in `UPPER_SNAKE_CASE`. Keep GUI logic in `src/gui.py` or adjacent modules and avoid heavy work at import time. No formatter or linter is configured, so keep changes readable and consistent with surrounding code.

## Testing Guidelines
There is no automated test suite or coverage target in this repo. Validate changes manually by running `python run.py` and exercising Discord login and rule matching flows. If you add automated tests, place them in a new `tests/` directory and document the runner you choose.

## Commit & Pull Request Guidelines
Recent commits use short, lowercase messages like `update`, so there is no formal convention. For new work, use descriptive, imperative commit summaries (for example `add rule filter`). Pull requests should include a clear summary, steps to test, notes about config changes, and screenshots for GUI updates.

## Security & Configuration Tips
Treat Discord tokens as secrets; keep them in `config/config.json` and do not commit them. Use `config/example_config.json` for shareable defaults.

---
> Source: [ZRnown/ironManDiscordReply](https://github.com/ZRnown/ironManDiscordReply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
