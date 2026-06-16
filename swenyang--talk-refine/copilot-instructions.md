## talk-refine

> TalkRefine is a Windows desktop speech-to-text tool. It uses FunASR (SenseVoice) for ASR and optionally an LLM for text refinement. The app runs as a system tray icon with an overlay UI, launched via a VBS script at startup.

# Copilot Agent Instructions

## Project Overview

TalkRefine is a Windows desktop speech-to-text tool. It uses FunASR (SenseVoice) for ASR and optionally an LLM for text refinement. The app runs as a system tray icon with an overlay UI, launched via a VBS script at startup.

## Development Environment

### Two deployment modes exist — be aware of the differences:

| | Dev machine (non-VM) | VM deployment |
|---|---|---|
| Python | System-wide install, no venv | venv (`venv/` directory) |
| Launch | Terminal / `python -m talkrefine` | VBS → `wscript.exe` → `pythonw.exe` (no console) |
| PATH | Single Python, `pip`/`python` always correct | venv NOT activated at runtime; `subprocess` finds **system** pip/python |

### ⚠️ Key Pitfall: subprocess + venv

When the app is launched via VBS (production mode on VM), the **venv is NOT activated**. This means:

- `import` works correctly (Python's `sys.path` points to venv site-packages)
- `subprocess.run(["pip", ...])` or `shutil.which("pip")` finds the **system Python's pip**, NOT the venv's pip

**Rule: Never use bare `"pip"` or `"python"` in subprocess calls.** If you must spawn a subprocess, use:
```python
import sys
subprocess.run([sys.executable, "-m", "pip", "install", ...])
```

### ⚠️ Key Pitfall: `trust_remote_code=True` in funasr

`funasr.AutoModel(trust_remote_code=True)` triggers `pip install -r requirements.txt` from the model snapshot on **every startup**. The model's `requirements.txt` includes heavy packages (torch, gradio, modelscope). This causes:

1. On dev machine: pip finishes quickly (packages already installed, same Python)
2. On VM: subprocess finds system pip → installs to wrong Python → hangs indefinitely

**Rule: Always use `trust_remote_code=False`** for SenseVoiceSmall. It uses standard funasr model format and does not need remote code execution. The `trust_remote_code` flag on the HuggingFace hub path only triggers pip install — it does NOT affect model loading.

## Build & Run

```powershell
# Activate venv (if using venv)
.\venv\Scripts\Activate.ps1

# Install in dev mode
pip install -e ".[win32,llamacpp]"

# Run
python -m talkrefine

# Install shortcuts & autostart
python -m talkrefine --install
```

## Project Structure

- `talkrefine/app.py` — Main application, hotkey handling, recording pipeline
- `talkrefine/asr/` — ASR engines (SenseVoice, Whisper)
- `talkrefine/llm/` — LLM providers (Ollama, llama.cpp)
- `talkrefine/ui/` — Overlay, tray icon, settings window
- `talkrefine/config.py` — Configuration loading
- `scripts/install.ps1` — One-click installer
- `scripts/start_hidden.vbs` — Silent launcher (no console window)

## Code Change Rules

1. **Do NOT commit or push** until the user has explicitly reviewed and approved changes.
2. After making code changes, restart the service and let the user verify before committing.
3. Test on venv + VBS launch path, not just terminal — many bugs only manifest in production launch mode.

---
> Source: [swenyang/talk-refine](https://github.com/swenyang/talk-refine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
