## yt-download-demo

> Operational guide for agentic coding assistants in this repository.

# AGENTS.md
Operational guide for agentic coding assistants in this repository.

## Repository Snapshot
- Project type: Python demo project for video/audio download and search.
- Python: `3.12` (`.python-version`, `pyproject.toml`).
- Dependency manager: `uv` (`uv.lock` is present).
- Primary libs: `yt-dlp`, `pytubefix` (from Git URLs).
- Flat source layout: `main.py`, `demo_ytdlp.py`, `demo_pytubefix.py`.
- No test suite exists yet.

## Rule Files
Checked for extra instruction files:
- `.cursorrules`: not found
- `.cursor/rules/`: not found
- `.github/copilot-instructions.md`: not found
If these files are added later, follow them when they conflict with this file.

## Build, Lint, and Test Commands
This repo has no formal build step. Use lint + script execution as verification.

### Environment / Dependencies
- Sync environment from lockfile:
  - `uv sync`
- Add dependency:
  - `uv add <package>`
- Run command inside project environment:
  - `uv run <command>`

### Run Scripts
- Basic yt-dlp demo:
  - `uv run python main.py`
- yt-dlp feature demo:
  - `uv run python demo_ytdlp.py`
- pytubefix feature demo:
  - `uv run python demo_pytubefix.py`

### Lint / Format
Ruff is configured in `pyproject.toml`.
- Lint all files:
  - `uv run ruff check .`
- Auto-fix lint issues:
  - `uv run ruff check . --fix`
- Format all files:
  - `uv run ruff format .`
- Verify formatting only:
  - `uv run ruff format . --check`

### Test Commands
No tests currently exist, but use pytest conventions when tests are added.
- Run all tests:
  - `uv run pytest`
- Run one test file:
  - `uv run pytest tests/test_example.py`
- Run one test function:
  - `uv run pytest tests/test_example.py::test_function_name`
- Run one class method:
  - `uv run pytest tests/test_example.py::TestClassName::test_method_name`

## Single-Test Execution (Important)
Preferred deterministic form (node id):
- `uv run pytest path/to/test_file.py::test_name`
Optional fuzzy matching while debugging:
- `uv run pytest -k "keyword"`
Prefer explicit node IDs for agent runs.

## Code Style: Enforced Rules
From `pyproject.toml`:
- Target Python: `py312`
- Line length: `119`
- Quote style: double quotes
- Indentation: spaces
- McCabe max complexity: `12`
- Enabled Ruff lint groups: `E`, `F`, `I`, `B`, `UP`, `C90`, `C4`, `SIM`, `RET`
- `E501` is ignored; keep long lines readable.

## Code Style: Local Conventions
Inferred from current source files.

### Imports
- Keep imports at module top.
- Rely on Ruff sorting (`I` rules).
- Remove unused imports.

### Naming
- Use `snake_case` for functions and variables.
- Use action-oriented function names (`demo_get_info`, `demo_audio_download`).
- Use clear config names (`ydl_opts`, `output_path`).

### Types
- Keep type hints on public params and returns.
- Prefer built-in types (`str`, `list[str]`, etc.) where practical.
- Avoid `Any` unless required by third-party APIs.

### Strings and Output
- Prefer f-strings for interpolation.
- Keep output simple and explicit.

### Entry Points and Resource Use
- Use `if __name__ == "__main__":` for script execution blocks.
- Use context managers for downloader lifecycle (`with yt_dlp.YoutubeDL(...)`).

### Error Handling
Current demo scripts mostly allow exceptions to surface.
For new non-trivial logic:
- Catch only exceptions you can handle.
- Include actionable context in errors (URL, path, operation).
- Never swallow exceptions silently.

### Comments and Docstrings
- Keep docstrings short and task-focused.
- Existing files use Chinese docstrings/comments; preserve local style unless asked to change.
- Add comments only for non-obvious behavior.

## Project Organization Guidance
- Keep changes minimal and targeted.
- Preserve current demo behavior unless task requires behavior changes.
- If reusable logic grows, create a package directory instead of expanding root scripts indefinitely.
- Keep generated artifacts in `outputs/` or dedicated data files, not in source modules.

## Runtime Notes
- Some flows rely on local `cookies.txt` for authenticated downloads.
- `demo_ytdlp.py` may depend on local runtime components (for example Node/EJS integration).
- Never commit secrets, cookies, or credentials.

## Agent Verification Checklist
After edits:
1. `uv run ruff check .`
2. `uv run ruff format . --check`
3. Run touched script(s) when feasible.
4. If tests exist for touched code, run targeted tests first, then full test run.
If a command cannot be executed, report why in handoff.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LePao1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LePao1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
