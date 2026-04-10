## ai-research-interview-lab

> This repository is a coding-interview practice workspace with a Tk/Ttk GUI for generating problem stubs, editing code, and running tests. Everything lives under `/Users/andrewzhao/Documents/coding_interview`.

# Project Context for Agents

This repository is a coding-interview practice workspace with a Tk/Ttk GUI for generating problem stubs, editing code, and running tests. Everything lives under `/Users/andrewzhao/Documents/coding_interview`.

## Key Components
- `tools/gui.py`: Main desktop GUI. Provides the in-app editor (smart indent/backspace, syntax highlighting), topic browser, stats pane, streak tracking overlay, test runner integration, and a collapsible Copilot (local LLM via Ollama) with streaming chat + Markdown rendering.
- `tools/generate_entry.py`: Registry of problems (`ProblemSpec`). Generates stubs/tests and backs the GUI & CLI topic list.
- `workspace/`: Auto-created package where generated solutions live. Files here run via `python -m workspace.<module>`.
- `ml/`, `coding/`, `leetcode/`: Reference implementations and topic material used for canonical solutions/tests.
- `Makefile`: Shortcuts (`make` / `make gui`) to launch the GUI.

## Typical Flows
1. Launch GUI via `python -m tools` (or `python tools/gui.py`).
2. Pick a topic from the left tree or use random/smart random.
3. `Generate` writes a stub into `workspace/` and opens it in the embedded editor.
4. Edit in place; save with `Cmd/Ctrl+S` or the `Save` button (red dot signals unsaved changes).
5. Run tests with the `Run Tests` button; output appears in the `Output` tab. Stats update automatically, streak counter animates on consecutive passes, and a celebratory popup appears on success.
6. Copilot: Toggle the right-side Copilot (`Copilot ▶` in header, or Cmd/Ctrl+Shift+C). Ask questions; it streams replies with Markdown (code fences, lists, tables, bold/italic). Context includes the current topic, your code, and the canonical implementation.
7. Need an external review? `Copy Review Prompt` (always enabled) pushes a comparison prompt to the clipboard and shows a quick “Prompt copied” toast.
8. Optional: view canonical implementation via `View Canonical`.

## Repo Conventions
- Python 3.10+, Tkinter available. No external dependencies assumed for GUI.
- Soft tabs set to 4 spaces in editor helpers.
- Generated stubs/tests should be deterministic and self-contained (no network calls).
- Stats persisted at `.practice_stats.json` in repo root; session temp files tracked in GUI.
- When adding topics, update `generate_entry.py` and, if needed, `gui.canonical_path_for_topic`.

## For Future Agents
- Prefer `rg` for searching; shell commands should be invoked with `["bash","-lc", ...]` and `workdir` set.
- Editing: use `apply_patch` for targeted changes; do not overwrite user content or undo unrelated modifications.
- GUI key-bindings are customized (including Cmd/Ctrl ± zoom that scales editor panes and the tree font); verify behavior analytically if you cannot run Tkinter.
- Before major changes, skim README and relevant files to understand current features.
- Tests are usually invoked per-problem via the GUI; if running manually, use `python -m workspace.<module>`.

### Copilot internals
- Backend: Ollama HTTP API (`/api/chat`) with streaming; keeps models warm via `keep_alive` and reuses KV via the returned `context`.
- UI: Built in `tools/gui.py` via `_build_copilot_panel` and toggled by `toggle_copilot`.
- Context assembly: `_assemble_copilot_context()` gathers topic, candidate code, and canonical content.
- Streaming: `_ollama_chat_stream()` handles newline-delimited JSON; updates Tk widgets via `after()`.
- Markdown: `_apply_markdown_to_range()` for headings, bold/italic, inline/fenced code, lists, blockquotes, and Markdown tables.
- Auto-start Ollama: `_maybe_start_ollama()` spawns `ollama serve` if found; controlled by `PRACTICE_AUTOSTART_OLLAMA` and `--no-llm-serve`.
- KV prewarm: `_maybe_prewarm_kv()` posts `num_predict: 0` with context at startup to reduce first-token latency; captured context is reused on next turns.

Caveats/notes
- Default model is `gpt-oss:20b`; on small RAM machines, advise switching to a 7B–14B model.
- If you modify context shape/contents, ensure `_assemble_copilot_context()` and prewarm behavior are kept in sync. Consider re‑prewarming on topic changes.
- Always call Tk updates from the main thread using `after()`.

## Useful Commands
- `python -m tools` — launch GUI.
- `python -m tools --no-llm-serve` — launch GUI without auto-starting Ollama.
- `python tools/generate_entry.py --list` — list available topics.
- `python tools/generate_entry.py --topic <id>` — generate stub/tests without GUI.
- `python -m workspace.<module>` — run a generated solution (tests included in file).

## Troubleshooting Notes
- GUI smart editing handlers live near the bottom of `tools/gui.py` (search for `_on_*`).
- Stats issues likely stem from `StatsManager` in the same file.
- Timers/daily goals handled inside GUI; ensure Tk event loop updates via `after`.
- If Tk bindings misbehave, check that `self.text.bind(...)` matches handler logic.
- Clipboard operations show transient toasts via `_show_prompt_copied`; if they stop appearing, confirm the Toplevel overlay is created (may fail on some WMs without display permissions).
- Copilot not responding: confirm Ollama is running (`ollama serve`). Status pill shows “Starting Ollama…” / “Ollama ready”. Set `OLLAMA_BASE_URL` if running on a non-default host/port.
- Slow first token: prewarm builds KV on launch; ensure keep-alive and KV reuse are active. Increase `keep_alive` if needed.

Keep this file updated when behaviors or workflows change so agents have immediate context.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Andrewzh112)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Andrewzh112)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
