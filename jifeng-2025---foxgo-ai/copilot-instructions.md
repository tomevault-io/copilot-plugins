## foxgo-ai

> FoxGo / 弈间 is a local 19x19 Go training website powered by KataGo, alongside

# Contributor and coding-agent guide

FoxGo / 弈间 is a local 19x19 Go training website powered by KataGo, alongside
the existing Windows FoxWQ screen-recognition overlay. User-facing text and
documentation are primarily Simplified Chinese.

## Architecture

- `training_server.py`: loopback-only HTTP API and static file server (Python standard library).
- `training/game.py`: authoritative rules, captures, positional superko, history, undo, SGF.
- `training/engine.py`: serialized KataGo analysis with timeouts and full move history.
- `training/analysis.cfg`: training-only settings, side-to-move evaluation perspective.
- `web/`: HTML/CSS/JavaScript frontend; no Node build step or external CDN.
- `app.py`, `vision.py`, `overlay.py`, `config.py`, `katago.py`, `board.py`: original Windows overlay.
- `bootstrap.py`: repeatable Windows engine/model setup, reuse installed files offline.

## Working rules

- Preserve both launchers and keep training independent of Windows UI dependencies.
- Enforce rules and turn ownership on the server; never trust the client board or AI move blindly.
- Send full history to KataGo. Snapshots alone cannot preserve ko/superko context.
- Keep evaluation perspective explicit; training uses SIDETOMOVE, legacy config may use BLACK.
- Difficulty presets are heuristic, not calibrated human ranks. Do not claim real kyu/dan strength.
- Preserve error recovery, revision checks, bounded waits, process cleanup and localhost protection.
- Two consecutive passes end play without authoritative dead-stone adjudication. Do not fabricate a result.
- Do not commit runtime binaries, personal config, virtual environments, logs or debug screenshots. User-approved documentation images under `docs/images/` should be tracked.
- Avoid changing user configuration or replacing installed models unnecessarily.
- Do not add a license or third-party binary redistribution without an owner decision.
- Keep responsive layout and keyboard controls usable; use native web assets for board and UI.
- Do not publish or configure a GitHub remote without a provided repository and authorization.

## Verification

Requires Python 3.10+. Training uses only the standard library.

```powershell
python -m unittest discover -s tests -v
python -m compileall -q training training_server.py bootstrap.py
node --check web/app.js
python training_server.py --port 8765
```

Run `python tests/smoke_katago.py` for integration with an installed engine/model.
This is separate from CI tests, which must run without GPU or network.
For UI changes, inspect desktop and narrow widths and verify a real move, AI reply,
top-three hints, undo and SGF export when an engine is available.
Legacy overlay checks require Windows and FoxWQ. Report when not performed; never
describe unit tests as proof of live screen-recognition behavior.

---
> Source: [jifeng-2025/-FoxGo-AI-](https://github.com/jifeng-2025/-FoxGo-AI-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
