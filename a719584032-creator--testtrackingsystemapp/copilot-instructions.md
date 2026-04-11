## testtrackingsystemapp

> `run.py` boots the desktop client and `build.py` orchestrates PyInstaller. Feature code lives in `services/`, `models/`, and `monitoring/` for backend I/O, domain objects, and hardware orchestration. `ui/` and `widgets/` hold all PyQt5 windows, with shared helpers in `utils/`. Defaults and file locations are defined in `config/`, static assets in `resources/`, and runtime artifacts should stay within `data/` or `logs/`.

# Repository Guidelines

## Project Structure & Module Organization
`run.py` boots the desktop client and `build.py` orchestrates PyInstaller. Feature code lives in `services/`, `models/`, and `monitoring/` for backend I/O, domain objects, and hardware orchestration. `ui/` and `widgets/` hold all PyQt5 windows, with shared helpers in `utils/`. Defaults and file locations are defined in `config/`, static assets in `resources/`, and runtime artifacts should stay within `data/` or `logs/`.

## Build, Test, and Development Commands
Set up a virtual environment before installing dependencies:
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```
Run the client locally with `python run.py`. Package a distributable with `python build.py` from the repository root; the script wires PyInstaller with the expected resource paths.

## Coding Style & Naming Conventions
Code should follow PEP 8 with 4-space indentation, type hints, and concise docstrings. Modules, packages, and functions use `snake_case`, while classes retain `CapWords`. Prefer explicit imports (`from services.api_client import ApiClient`) and mirror the module-level `logger = logging.getLogger(__name__)` pattern found in `services/api_client.py`. Run `python -m compileall .` or your editor’s formatter before submitting; Black-compatible formatting is encouraged but not enforced.

## Commit & Pull Request Guidelines
Commit messages use sentence-case imperatives (`Ensure monitoring message loops stop on manual abort`), so keep them short and actionable. Squash noisy intermediates before opening a PR. PRs should describe the problem, the solution approach, and UI impacts; attach logs or screenshots for visual changes and link to tracking issues. Confirm local packaging (`python build.py`) for release-facing updates and note any manual steps reviewers must execute.

## Security & Configuration Tips
Never commit real credentials or tokens—use the encrypted helpers under `services/auth.py` and sanitize logs before sharing. When changing defaults, update `config/settings.py` and document new environment variables in the PR description. For OTA adjustments, ensure paths align with the `PATVS_ROOT` logic so installers remain portable across Windows and Linux.

## Core Prompt Requirements(Product Rules)
This Windows client monitors test case execution:
1. When a test case runs, the system **auto-selects and enables monitoring options** based on the case **keywords** (processes, services, hardware, network, logs, etc.).
2. A test case can be marked **Pass only after all monitoring actions have completed**; users may mark **Fail** or **Blocked** at any time.
3. Once the final result is submitted, the client must **stop all active monitors** and release resources (processes/threads/handles/file handles/timers, etc.).
4. The run may include **shutdowns/reboots**. On the next client start, the app must **automatically restore** the **previous execution record and monitoring state**, then continue or clean up as appropriate:
   - Restore unsubmitted cases, monitor toggles, timers, and collected evidence (logs, screenshots, metrics).
   - If resuming safely is impossible, prompt for manual intervention and provide a “one-click cleanup & retry”.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/a719584032-creator)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/a719584032-creator)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
