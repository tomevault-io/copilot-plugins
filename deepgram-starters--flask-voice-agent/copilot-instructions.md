## gitignore-requirements

> Gitignore File Requirements

# Gitignore Requirements

## Specifications
- MUST ignore all the usual files and folders including:

# environment artifacts
.venv
.env
venv/
venv.bak/
.vscode/
.DS_Store
Pipfile
Pipfile.lock

# python artifacts
__pycache__
*.egg-info
dist/
.mypy_cache/
.pytest_cache/

# build
build/
poetry.lock
dist

# examples
chatlog.txt
output_*.wav

---
> Source: [deepgram-starters/flask-voice-agent](https://github.com/deepgram-starters/flask-voice-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
