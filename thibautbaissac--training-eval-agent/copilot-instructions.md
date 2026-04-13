## training-eval-agent

> Auto-generated from all feature plans. Last updated: 2026-04-03

# training-eval-agent Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-04-03

## Active Technologies

- Python 3.10+ (standalone application)
- Anthropic Python SDK (`anthropic`) — direct API calls to Claude
- Baileys Node.js bridge (bundled) — WhatsApp Web transport
- websockets, pyyaml, click
- File-based storage (YAML/JSON + JSONL sessions) — no database

## Project Structure

```text
src/                  # Python application (agent loop, tools, evaluator, scorer, reports)
bridge/               # Node.js WhatsApp bridge (baileys)
workspace/            # Markdown-driven agent config (SOUL.md, AGENTS.md, skills/)
config/               # settings.yaml
manifests/            # Training manifest YAML files
data/                 # Runtime state — evaluations, sessions, reports (gitignored)
tests/                # pytest test suite
specs/                # Feature specifications
```

## Commands

```bash
pip install -e .                                  # Install project
cd bridge && npm install && npm run build          # Build bridge
cd bridge && npm start                             # Start WhatsApp bridge (QR code)
python -m src.main launch --manifest <path>        # Launch evaluation
python -m src.main status --session <id>           # Check status
python -m src.main report --session <id> --employee <name>  # Individual report
python -m src.main summary --session <id>          # Session summary
python -m src.main purge                           # Purge old data
pytest                                             # Run tests
```

## Architecture

Standalone Python app inspired by Nanobot's patterns but with NO dependency on `nanobot-ai`:
- Own agent loop using Anthropic Messages API with tool use
- Baileys bridge for WhatsApp (bundled Node.js, same protocol as Nanobot)
- Markdown-driven agent config (SOUL.md, AGENTS.md, SKILL.md)
- File-based state (YAML/JSON, JSONL sessions) — no database
- Two-layer memory (MEMORY.md facts + HISTORY.md events)
- asyncio heartbeat for nudge scheduling

## Recent Changes

- 001-training-eval-agent: Standalone architecture — own agent loop, bundled bridge, file-based storage

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThibautBaissac)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ThibautBaissac)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
