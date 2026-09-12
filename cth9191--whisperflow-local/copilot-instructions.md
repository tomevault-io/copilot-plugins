## whisperflow-local

> This is a 100% local WhisperFlow clone for Windows: tap `Ctrl+Win`, speak, and

# whisperflow-local — guide for Claude Code

This is a 100% local WhisperFlow clone for Windows: tap `Ctrl+Win`, speak, and
cleaned text is pasted into the focused field. STT runs on `faster-whisper`
(CUDA); cleanup runs on a local Ollama model. No cloud at dictation time.

When a user asks you to **set this up on their machine**, run the setup flow below
and fix failures until `doctor` passes. When they ask how it works, point them at
`PLAN.md` and `docs/architecture.html`.

## Setup flow (run in order)

1. **Python env + deps** — requires Python 3.11+ and `uv`:
   ```bash
   uv venv --python 3.11
   uv pip install -e .
   ```
   If `uv` is missing, install it first (https://docs.astral.sh/uv/).

2. **Ollama + cleanup model** — the cleanup LLM runs on Ollama:
   - Confirm Ollama is installed (`ollama --version`) and running. If not,
     point the user to https://ollama.com/download (it installs as an
     auto-starting service).
   - Read `cleanup.model` from `config.yaml`, then `ollama pull <that model>`
     (currently `granite4.1:3b`). The model is small and kept warm via
     `keep_alive` so dictation latency stays low.

3. **Validate** — run the doctor and resolve every failing check:
   ```bash
   .venv/Scripts/python.exe -m whisperflow_local doctor
   ```
   It checks microphone, CUDA, Ollama (+ model present, loopback-only),
   clipboard, and the hotkey/overlay imports.

4. **Run it**:
   ```bash
   .venv/Scripts/python.exe -m whisperflow_local run
   ```
   For always-on, `... install-autostart` writes a windowless Startup shortcut.

## Known gotchas (fix these when doctor fails)

- **CUDA / `cuda` check fails** — `faster-whisper` (via `ctranslate2`) needs the
  cuDNN/cuBLAS DLLs on the path. On Windows the usual fix is installing the
  matching NVIDIA cuDNN, or `pip install nvidia-cudnn-cu12 nvidia-cublas-cu12`
  into the venv so the DLLs ship alongside. If no usable GPU, set
  `stt.device: "cpu"` in `config.yaml` (much slower) and warn the user.
- **`ollama` check fails / model MISSING** — Ollama service not running, or the
  model in `config.yaml` hasn't been pulled. Pull it, or change `cleanup.model`
  to one the user already has (`ollama list`).
- **Hotkey does nothing** — `Ctrl+Win` is the default. It's a modifier pair with
  no built-in OS action (Wispr Flow's approach), so it shouldn't collide. If the
  user wants a different key, edit `hotkey.combo` in `config.yaml`. Avoid
  reserved single `Win+<letter>` combos (e.g. Win+F opens Feedback Hub).
- **Paste lands in the wrong app / nothing** — insertion re-verifies focus; if
  focus moved, it aborts and stashes the text on the clipboard. Elevated/RDP/
  anti-cheat windows reject synthetic input (documented limitation).

## Commands

| Command | What |
|---|---|
| `doctor` | environment health check |
| `run` | live app (tray + hotkey) |
| `selftest` | autonomous end-to-end on a bundled spoken sample |
| `install-autostart` / `uninstall-autostart` | launch on login (windowless) |

Always invoke via the venv interpreter (`.venv/Scripts/python.exe`) so the right
deps are used.

## Layout

`whisperflow_local/` — `app.py` (orchestrator + state machine), `audio.py`
(capture), `stt.py`, `cleanup.py`, `inserter.py` (focus-safe paste), `overlay.py`
(the pill), `tray.py`, `autostart.py`, `hotkey.py`, `sound.py`, `config.py`.
Config is `config.yaml`. `tools/` renders the icon. `docs/architecture.html` is
the visual explainer.

---
> Source: [cth9191/whisperflow-local](https://github.com/cth9191/whisperflow-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
