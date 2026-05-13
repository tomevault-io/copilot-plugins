## recall

> - `src/` holds the Python packages that power Recall: `detector/` for meeting signals, `recorder/` for ffmpeg capture, `transcription/` for Whisper, `summarization/` for Ollama prompts, `orchestrator/` for the daemon, and `storage/` for retention logic. Keep new modules grouped with their nearest peer.

# Repository Guidelines

## Project Structure & Module Organization
- `src/` holds the Python packages that power Recall: `detector/` for meeting signals, `recorder/` for ffmpeg capture, `transcription/` for Whisper, `summarization/` for Ollama prompts, `orchestrator/` for the daemon, and `storage/` for retention logic. Keep new modules grouped with their nearest peer.
- `browser-extension/` contains the Chrome extension and native messaging bridge—update both sides when changing event schemas.
- `config/` and `config/config.yaml` define device names, model choices, and extension ports; never hard-code duplicates in code.
- `data/` stores runtime recordings and outputs; treat it as disposable and do not check in artifacts.
- `scripts/` provides operational helpers (setup, daemon management, diagnostics); extend existing scripts rather than adding top-level shell commands.
- Place documentation in `docs/` and automated checks or fixtures in `tests/`.

## Build, Test, and Development Commands
- `./scripts/setup.sh` installs Homebrew dependencies, boots the virtualenv, and pulls the default LLM—run once on new machines.
- Manual Python setup: `python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt`.
- Launch the daemon locally with `python -m src.orchestrator.daemon`; monitor it with `./scripts/status.sh` and tail `data/logs/daemon.log`.
- Hardware sanity checks live in `./scripts/test_components.sh`; diarization model validation is `python scripts/test_diarization.py`.
- Use `./scripts/install_daemon.sh` and `./scripts/uninstall_daemon.sh` when testing launchd behavior.

## Coding Style & Naming Conventions
- Target Python 3.9+ with 4-space indentation, PEP 8 line lengths, and type hints (see `src/orchestrator/daemon.py` for tone).
- Modules and functions use `snake_case`, classes use `PascalCase`, and configuration keys mirror the YAML naming already shipped.
- Prefer explicit imports from sibling packages and keep user-facing strings centralized in config where practical.
- No formatter is enforced; if you run one (e.g., `black`), ensure the diff stays localized.

## Testing Guidelines
- New automated tests belong under `tests/` and should run without audio hardware; prefer dependency injection or mocks for ffmpeg, Ollama, and Whisper boundaries.
- Name files `test_<feature>.py` and structure cases with `unittest` or `pytest` (install locally as needed).
- For hardware or model regressions, wire lightweight smoke checks into `scripts/test_components.sh` rather than creating ad-hoc scripts.
- Document any manual verification steps in the PR description, especially those touching Audio MIDI Setup or Ollama models.

## Commit & Pull Request Guidelines
- Commit messages should be imperative and scoped by component, e.g., `recorder: improve device fallback`. Group related changes together and keep subject lines under ~72 characters.
- PRs must describe the feature, note config or audio-device impacts, list commands executed (e.g., tests or scripts), and attach screenshots/log snippets for UI or daemon output changes.
- Link to tracked issues when available; if none exist, provide reproduction and acceptance notes inline.

## Security & Configuration Tips
- Never commit real recordings, transcripts, or logs; verify `.gitignore` covers anything added under `data/`.
- Treat `config/*.yaml` as user-editable: ship safe defaults, gate secrets behind environment variables, and call out any migration steps in release notes.

---
> Source: [trevhud/recall](https://github.com/trevhud/recall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
