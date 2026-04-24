## aidriven

> Auto-generated from all feature plans. Last updated: 2026-04-07

# aidriven Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-04-07

## Active Technologies
- Python 3.11+ (dev: 3.14) + Python stdlib only — `urllib.request`, `urllib.error`, `ssl`, `json`, `tarfile`, `hashlib`, `pathlib`, `shutil`, `os`, `tempfile`, `dataclasses`, `enum`, `logging`, `re`, `argparse`, `platform`, `sys`. (No third-party runtime deps; aligns with constitution §VII "minimal dependencies".) (v0.2.0/spec-002-cli-artifact-install)
- Local filesystem only. Project lockfile: `<project-root>/aidriven-lock.json`. User lockfile: `~/.cache/aidriven/install-records.json`. Cache: OS user cache directory (`~/.cache/aidriven/cache/` on Linux/macOS, `%LOCALAPPDATA%\aidriven\Cache\` on Windows). (v0.2.0/spec-002-cli-artifact-install)

- Python 3.11+ (dev: 3.14) + None (stdlib only — `pathlib`, `shutil`, `platform`, `subprocess`, `dataclasses`, `enum`, `logging`, `abc`) (v0.1.0/spec-001-ide-discovery)

## Project Structure

```text
src/
tests/
```

## Commands

cd src [ONLY COMMANDS FOR ACTIVE TECHNOLOGIES][ONLY COMMANDS FOR ACTIVE TECHNOLOGIES] pytest [ONLY COMMANDS FOR ACTIVE TECHNOLOGIES][ONLY COMMANDS FOR ACTIVE TECHNOLOGIES] ruff check .

## Code Style

Python 3.11+: Follow standard conventions

## Recent Changes
- v0.2.0/spec-002-cli-artifact-install: Added Python 3.11+ (dev: 3.14) + Python stdlib only — `urllib.request`, `urllib.error`, `ssl`, `json`, `tarfile`, `hashlib`, `pathlib`, `shutil`, `os`, `tempfile`, `dataclasses`, `enum`, `logging`, `re`, `argparse`, `platform`, `sys`. (No third-party runtime deps; aligns with constitution §VII "minimal dependencies".)

- v0.1.0/spec-001-ide-discovery: Added Python 3.11+ + None (stdlib only — `pathlib`, `shutil`, `platform`, `subprocess`, `dataclasses`, `enum`, `logging`, `abc`)

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThiagoPanini) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
