## openai-sublime-text

> Project-specific notes for working on the Sublime Text plugin in this repo.

# AGENTS.md

Project-specific notes for working on the Sublime Text plugin in this repo.

## Plugin loading

- `main.py` is the plugin registration hub.
- New `TextCommand`, `WindowCommand`, and `EventListener` classes must be imported in `main.py`, or Sublime may not register them even if the module is imported indirectly.
- If a shortcut or command "does nothing", check `main.py` first before debugging the command body.

## OpenAI input panel

- The replacement multiline input lives in `plugins/input_panel.py`.
- The panel name is `output.openai_input`.
- Submit is triggered from the panel via `openai_submit_input_panel_from_view`, which delegates to `openai_submit_input_panel`.
- Keep the panel empty on open unless there is an explicit reason to prefill it.

## Selection handling

- The repo already collects selected text into the request pipeline in `plugins/openai_base.py`.
- Do not also copy selection into the input panel unless you intentionally want duplicated context.
- If you change selection behavior, verify both the panel contents and the final LLM payload path.

## Key bindings

- Plugin key bindings can be overridden by user-level Sublime key bindings.
- If a package shortcut does not fire, inspect user keymaps before assuming the plugin command is broken.
- For panel-specific input behavior, keymaps alone may be insufficient; intercepting commands via an `EventListener.on_text_command()` hook can be more reliable.

- `Cmd+Enter` support for this panel depends on the panel-scoped bindings in `Default.sublime-keymap`.
- When debugging key bindings, compare against the sibling `CodexSublime` project; its panel behavior is the reference implementation for this feature.

## Debugging

- When debugging Sublime/plugin issues from the shell, check macOS unified logs for `Sublime Text` and `plugin_host`, for example:
  - `/usr/bin/log show --style compact --last 20m --predicate '(process == "Sublime Text") OR (process CONTAINS "plugin_host")'`
- System logs are useful for crashes and load failures, but not every command/key-binding issue shows up there.
- If a command appears missing, also inspect the Sublime console inside the app for `unknown command`, import errors, and plugin tracebacks.

## Verification

- After changing plugin Python files, run the smallest syntax check that covers the touched files, for example:
  - `python3 -m compileall main.py plugins/input_panel.py`
  - `python3 -m compileall plugins/openai_base.py`
- If Sublime appears to ignore a new command, listener, or key binding after code changes, reload the plugin or restart Sublime Text before assuming the code path is wrong.

---
> Source: [yaroslavyaroslav/OpenAI-sublime-text](https://github.com/yaroslavyaroslav/OpenAI-sublime-text) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
