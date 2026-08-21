## cross-application-automation-and-execution-agent-based-on-vlm

> Desktop GUI automation agent backed by a VLM, with two execution modes:

# AGENTS.md — Qwen-Agent (Desktop GUI Automation)

## Project overview

Desktop GUI automation agent backed by a VLM, with two execution modes:
- **Terminal interactive**: `run_gui_owl_1_5_for_pc.py` — REPL via `input()`.
- **Electron bridge**: `agent_bridge.py` — JSON Lines IPC, called by the `desktop/` Electron app.
Shared logic: `utils.py`. All agents use LangGraph (`StateGraph`).

## How to run

### Terminal mode (standalone)
```powershell
python run_gui_owl_1_5_for_pc.py
```
Uses hardcoded `API_KEY` / `BASE_URL` / `MODEL_NAME` constants at the top of the file. Interactive prompt loop.

### Electron desktop app
```powershell
cd desktop
npm install
npm run dev
```
The app spawns `agent_bridge.py` as a child process. API key is passed via `AGENT_API_KEY` env var (never CLI args). See `desktop/src/main/python-runner.ts:50-57`.

### Bridge mode (debug / manual)
```powershell
$env:AGENT_API_KEY = "sk-xxx"
python agent_bridge.py `
    --instruction "task" `
    --base-url "https://dashscope.aliyuncs.com/compatible-mode/v1" `
    --model-name "qwen3-vl-plus" `
    --output-dir "C:\Users\...\Desktop\anno" `
    --max-steps 50
```
Outputs JSON Lines on stdout for real-time consumption.

## Architecture

- **VLM backend**: `GUIOwlWrapper` (`utils.py:767`) wraps the OpenAI-compatible chat API.
- **Message construction**: `build_messages()` builds DashScope-format message lists; `GUIOwlWrapper.convert_messages_format_to_openaiurl()` translates them to OpenAI format, converting images to base64 data URIs via `image_to_base64()`.
- **Image resize**: `smart_resize()` (`utils.py:394`) resizes screenshots to factor-aligned dimensions before sending to the VLM. Default factor=16, min_pixels=3136, max_pixels=10035200 * 200 in the agent (much larger default than `image_to_base64`'s 10035200 for consistency).
- **Coordinate system**: model outputs 0–1000 normalized coords; `rescale_coordinates()` converts to actual pixels using resized screenshot dimensions from `smart_resize()`. Note: the agent graph uses `resize_node` with huge `max_pixels=1003520*200`, so coordinate rescaling uses the *resized* dimensions, not original screen pixels.
- **LangGraph execution loop**: `observe → build_messages → plan → parse → resize → act → update_history → loop`. Screenshot capture, VLM call, tool-call extraction, coordinate rescaling, action execution, 2s wait, repeat.
- **agent_bridge.py monkey-patches** `core.MODEL_NAME` (`agent_bridge.py:177`) so `build_messages()` uses the user's model name, not the hardcoded constant.

## Critical gotchas

- **Hardcoded API key in `run_gui_owl_1_5_for_pc.py:28`**: the terminal entry point bakes in a real API key and base URL. The Electron bridge (`agent_bridge.py`) is the production path — it reads config from env/cli args. If working on the terminal mode, either replace the constants or pass `--api_key` etc. (currently unsupported — the constants are module-level).
- **Do NOT call `core.main()` from bridge code** — it uses `input()` and hardcoded config. Always create your own `GUIOwlWrapper` and call `core.run_agent()` directly (see `agent_bridge.py:179-197`).
- **`max_pixels` mismatch**: `resize_node` in the agent graph passes `max_pixels=1003520*200` to `smart_resize()`, but `image_to_base64()` in utils uses `max_pixels=10035200`. This means the screenshot sent to the VLM is resized differently from the dimensions used for coordinate rescaling. If coordinates seem off, check that both paths use the same resize parameters.
- **`stdout` redirect in `agent_bridge.py`**: Python stdout is replaced with `_JsonLinesStream` to emit JSON Lines. Use `sys.__stdout__` to bypass. Stderr is not redirected — Electron reads stderr separately.
- **UTF-8 on Windows**: `agent_bridge.py:33-37` forces `utf-8` on all stdio handles for CJK support.

## Dependencies (no lockfile or pyproject.toml)

Python:
- `pyautogui`, `pyperclip` — desktop automation
- `PIL` (Pillow) — screenshots and annotation
- `numpy` — type hints in abstract classes
- `openai` — VLM client
- `langgraph` — agent execution graph
- `tkinter` — StepPopup UI (bundled with Python on Windows)

Node (desktop app only):
- `npm install` in `desktop/`
- `npm run dev` — Electron + Vite dev server
- `npm run build` — production build

---
> Source: [Oxygen-Bobo/Cross-Application-Automation-and-Execution-Agent-Based-on-VLM](https://github.com/Oxygen-Bobo/Cross-Application-Automation-and-Execution-Agent-Based-on-VLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
